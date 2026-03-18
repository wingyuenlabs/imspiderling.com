---
Title: Claude CodeでUnit of Workパターンを設計する：複数リポジトリのトランザクション統一・変更追跡・原子コミット
Description: 
Author: myougaTheAxo
Date: 2026-03-18T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  はじめに
</h2>

<p>「複数のリポジトリを使う処理でトランザクションが統一されていなかった」——Unit of Workパターンで複数リポジトリへの変更を1つのトランザクションにまとめ、原子的にコミット/ロールバックする設計をClaude Codeに生成させる。</p>




<h2>
  
  
  CLAUDE.mdにUnit of Work設計ルールを書く
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Unit of Workパターン設計ルール</span>

<span class="gu">### 役割</span>
<span class="p">-</span> 1つのユースケース（トランザクション）スコープを管理
<span class="p">-</span> スコープ内で取得した集約を追跡（変更検出）
<span class="p">-</span> commit()で全変更を一括保存、rollback()で全変更を取り消し

<span class="gu">### リポジトリとの関係</span>
<span class="p">-</span> リポジトリはUoWが管理するDBコネクション/Prismaトランザクションを使用
<span class="p">-</span> UoWを通じてリポジトリを取得する（UoW.getOrderRepository()）
<span class="p">-</span> UoW外でリポジトリを直接使用しない

<span class="gu">### Prismaとの統合</span>
<span class="p">-</span> Prisma.$transaction(async (tx) =&gt; { ... }) をUoWのスコープとして使用
<span class="p">-</span> リポジトリはPrismaトランザクション（tx）を受け取る
</code></pre>

</div>






<h2>
  
  
  Unit of Work実装の生成
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">Unit</span> <span class="k">of</span> <span class="nx">Workパターンを設計してください</span><span class="err">。</span>

<span class="nx">要件</span><span class="err">：</span>
<span class="o">-</span> <span class="nx">Prismaトランザクションとの統合</span>
<span class="o">-</span> <span class="nx">リポジトリアクセスのファクトリー</span>
<span class="o">-</span> <span class="nx">ドメインイベントの収集とコミット後の発行</span>
<span class="o">-</span> <span class="nx">エラー時の自動ロールバック</span>

<span class="nx">生成ファイル</span><span class="p">:</span> <span class="nx">src</span><span class="o">/</span><span class="nx">infrastructure</span><span class="o">/</span><span class="nx">unitOfWork</span><span class="o">/</span>
</code></pre>

</div>






<h2>
  
  
  生成されるUnit of Work実装
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/infrastructure/unitOfWork/unitOfWork.ts — Unit of Work</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">IUnitOfWork</span> <span class="p">{</span>
  <span class="nl">orderRepository</span><span class="p">:</span> <span class="nx">IOrderRepository</span><span class="p">;</span>
  <span class="nl">userRepository</span><span class="p">:</span> <span class="nx">IUserRepository</span><span class="p">;</span>
  <span class="nl">inventoryRepository</span><span class="p">:</span> <span class="nx">IInventoryRepository</span><span class="p">;</span>
  <span class="nf">commit</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nf">rollback</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">PrismaUnitOfWork</span> <span class="k">implements</span> <span class="nx">IUnitOfWork</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">_orderRepository</span><span class="p">:</span> <span class="nx">IOrderRepository</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_userRepository</span><span class="p">:</span> <span class="nx">IUserRepository</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_inventoryRepository</span><span class="p">:</span> <span class="nx">IInventoryRepository</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">_aggregates</span><span class="p">:</span> <span class="nx">AggregateRoot</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="k">private</span> <span class="nx">_committed</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="k">readonly</span> <span class="nx">tx</span><span class="p">:</span> <span class="nx">PrismaTransactionClient</span><span class="p">)</span> <span class="p">{}</span>

  <span class="kd">get</span> <span class="nf">orderRepository</span><span class="p">():</span> <span class="nx">IOrderRepository</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_orderRepository</span> <span class="o">??=</span> <span class="k">new</span> <span class="nc">PrismaOrderRepository</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">tx</span><span class="p">,</span> <span class="k">this</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_orderRepository</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">get</span> <span class="nf">userRepository</span><span class="p">():</span> <span class="nx">IUserRepository</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_userRepository</span> <span class="o">??=</span> <span class="k">new</span> <span class="nc">PrismaUserRepository</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">tx</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_userRepository</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">get</span> <span class="nf">inventoryRepository</span><span class="p">():</span> <span class="nx">IInventoryRepository</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_inventoryRepository</span> <span class="o">??=</span> <span class="k">new</span> <span class="nc">PrismaInventoryRepository</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">tx</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_inventoryRepository</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="c1">// リポジトリが集約を登録（変更追跡用）</span>
  <span class="nf">registerAggregate</span><span class="p">(</span><span class="nx">aggregate</span><span class="p">:</span> <span class="nx">AggregateRoot</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_aggregates</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">aggregate</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">commit</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_committed</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Unit of Work already committed</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_committed</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
    <span class="c1">// コミット自体はPrismaトランザクションが行う（tx.commit相当）</span>
    <span class="c1">// ドメインイベントを収集</span>
    <span class="kd">const</span> <span class="nx">events</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">_aggregates</span><span class="p">.</span><span class="nf">flatMap</span><span class="p">(</span><span class="nx">a</span> <span class="o">=&gt;</span> <span class="nx">a</span><span class="p">.</span><span class="nf">collectDomainEvents</span><span class="p">());</span>

    <span class="c1">// コミット後のイベント発行はスコープ外で行う</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_pendingEvents</span> <span class="o">=</span> <span class="nx">events</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">private</span> <span class="nx">_pendingEvents</span><span class="p">:</span> <span class="nx">DomainEvent</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="nf">getPendingEvents</span><span class="p">():</span> <span class="nx">DomainEvent</span><span class="p">[]</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_pendingEvents</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">rollback</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="c1">// Prismaトランザクションのロールバックはthrowで発動</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_aggregates</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_pendingEvents</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// UoWファクトリー</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">UnitOfWorkFactory</span> <span class="p">{</span>
  <span class="k">static</span> <span class="k">async</span> <span class="nx">run</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">fn</span><span class="p">:</span> <span class="p">(</span><span class="nx">uow</span><span class="p">:</span> <span class="nx">PrismaUnitOfWork</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">afterCommit</span><span class="p">?:</span> <span class="p">(</span><span class="nx">events</span><span class="p">:</span> <span class="nx">DomainEvent</span><span class="p">[])</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span>
  <span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="na">pendingEvents</span><span class="p">:</span> <span class="nx">DomainEvent</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">prisma</span><span class="p">.</span><span class="nf">$transaction</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">tx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">uow</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">PrismaUnitOfWork</span><span class="p">(</span><span class="nx">tx</span> <span class="k">as</span> <span class="kr">any</span><span class="p">);</span>
      <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fn</span><span class="p">(</span><span class="nx">uow</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nf">commit</span><span class="p">();</span>
      <span class="nx">pendingEvents</span> <span class="o">=</span> <span class="nx">uow</span><span class="p">.</span><span class="nf">getPendingEvents</span><span class="p">();</span>
      <span class="k">return</span> <span class="nx">result</span><span class="p">;</span>
    <span class="p">});</span>

    <span class="c1">// トランザクション成功後にドメインイベントを発行</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">pendingEvents</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">afterCommit</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nf">afterCommit</span><span class="p">(</span><span class="nx">pendingEvents</span><span class="p">).</span><span class="k">catch</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span>
        <span class="nx">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">({</span> <span class="nx">err</span> <span class="p">},</span> <span class="dl">'</span><span class="s1">Post-commit event dispatch failed</span><span class="dl">'</span><span class="p">)</span>
      <span class="p">);</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="nx">result</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/infrastructure/repositories/orderRepository.ts — UoW対応リポジトリ</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">PrismaOrderRepository</span> <span class="k">implements</span> <span class="nx">IOrderRepository</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="nx">tx</span><span class="p">:</span> <span class="nx">PrismaTransactionClient</span><span class="p">,</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="nx">uow</span><span class="p">:</span> <span class="nx">PrismaUnitOfWork</span>
  <span class="p">)</span> <span class="p">{}</span>

  <span class="k">async</span> <span class="nf">findById</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Order</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">row</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">tx</span><span class="p">.</span><span class="nx">order</span><span class="p">.</span><span class="nf">findUnique</span><span class="p">({</span>
      <span class="na">where</span><span class="p">:</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">},</span>
      <span class="na">include</span><span class="p">:</span> <span class="p">{</span> <span class="na">items</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
    <span class="p">});</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">row</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="nx">OrderMapper</span><span class="p">.</span><span class="nf">toDomain</span><span class="p">(</span><span class="nx">row</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">uow</span><span class="p">.</span><span class="nf">registerAggregate</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span> <span class="c1">// 変更追跡に登録</span>
    <span class="k">return</span> <span class="nx">order</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">save</span><span class="p">(</span><span class="nx">order</span><span class="p">:</span> <span class="nx">Order</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">tx</span><span class="p">.</span><span class="nx">order</span><span class="p">.</span><span class="nf">upsert</span><span class="p">({</span>
      <span class="na">where</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">id</span> <span class="p">},</span>
      <span class="na">create</span><span class="p">:</span> <span class="nx">OrderMapper</span><span class="p">.</span><span class="nf">toCreateData</span><span class="p">(</span><span class="nx">order</span><span class="p">),</span>
      <span class="na">update</span><span class="p">:</span> <span class="nx">OrderMapper</span><span class="p">.</span><span class="nf">toUpdateData</span><span class="p">(</span><span class="nx">order</span><span class="p">),</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// アプリケーション層: UoWを使ったユースケース</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">PlaceOrderUseCase</span> <span class="p">{</span>
  <span class="k">async</span> <span class="nf">execute</span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="nx">PlaceOrderInput</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Order</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">UnitOfWorkFactory</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span>
      <span class="k">async </span><span class="p">(</span><span class="nx">uow</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="c1">// 同一トランザクション内で複数リポジトリを操作</span>
        <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nx">userRepository</span><span class="p">.</span><span class="nf">findById</span><span class="p">(</span><span class="nx">input</span><span class="p">.</span><span class="nx">userId</span><span class="p">);</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">NotFoundError</span><span class="p">(</span><span class="dl">'</span><span class="s1">User not found</span><span class="dl">'</span><span class="p">);</span>

        <span class="c1">// 在庫確認</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">item</span> <span class="k">of</span> <span class="nx">input</span><span class="p">.</span><span class="nx">items</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">inventory</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nx">inventoryRepository</span><span class="p">.</span><span class="nf">findByProductId</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">productId</span><span class="p">);</span>
          <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">inventory</span><span class="p">.</span><span class="nf">hasEnough</span><span class="p">(</span><span class="nx">Quantity</span><span class="p">.</span><span class="k">of</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">quantity</span><span class="p">,</span> <span class="dl">'</span><span class="s1">piece</span><span class="dl">'</span><span class="p">)))</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">OutOfStockError</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">productId</span><span class="p">);</span>
          <span class="p">}</span>
        <span class="p">}</span>

        <span class="c1">// 注文作成</span>
        <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="nx">Order</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span><span class="nx">input</span><span class="p">.</span><span class="nx">userId</span><span class="p">,</span> <span class="nx">input</span><span class="p">.</span><span class="nx">items</span><span class="p">);</span>

        <span class="c1">// 在庫を減らす</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">item</span> <span class="k">of</span> <span class="nx">input</span><span class="p">.</span><span class="nx">items</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">inventory</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nx">inventoryRepository</span><span class="p">.</span><span class="nf">findByProductId</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">productId</span><span class="p">);</span>
          <span class="nx">inventory</span><span class="p">.</span><span class="nf">decrement</span><span class="p">(</span><span class="nx">Quantity</span><span class="p">.</span><span class="k">of</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">quantity</span><span class="p">,</span> <span class="dl">'</span><span class="s1">piece</span><span class="dl">'</span><span class="p">));</span>
          <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nx">inventoryRepository</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="nx">inventory</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="c1">// 注文保存</span>
        <span class="k">await</span> <span class="nx">uow</span><span class="p">.</span><span class="nx">orderRepository</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span>

        <span class="k">return</span> <span class="nx">order</span><span class="p">;</span>
        <span class="c1">// commit()は自動（UnitOfWorkFactory.runが呼ぶ）</span>
      <span class="p">},</span>
      <span class="c1">// コミット後にドメインイベントを発行</span>
      <span class="k">async </span><span class="p">(</span><span class="nx">events</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">eventDispatcher</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="nx">events</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  まとめ
</h2>

<p>Claude CodeでUnit of Workパターンを設計する：</p>

<ol>
<li>
<strong>CLAUDE.md</strong> にUoWがPrismaトランザクションをスコープとして管理・リポジトリはUoWから取得・コミット後にドメインイベントを発行・エラーはthrowでロールバックを明記</li>
<li>
<strong><code>UnitOfWorkFactory.run(tx =&gt; { ... })</code></strong> でユースケース全体を1トランザクションに——注文作成・在庫更新・ユーザーポイント更新を全て同一<code>tx</code>で実行してアトミックに成功/失敗</li>
<li>
<strong><code>registerAggregate</code>で変更追跡</strong> し、commit()時にドメインイベントを収集——「どの集約が変更されたか」をUoWが把握してイベント収集を自動化</li>
<li>
<strong>コミット後のイベント発行</strong> で「DBは成功したがイベントが出なかった」問題を防止——トランザクション外でのイベント発行でロールバック済みイベントが流れることもない</li>
</ol>




<p><em>アーキテクチャ設計のレビューは **Code Review Pack（¥980）</em>* の <code>/code-review</code> で確認できます。*</p>

<p><em><a href="https://prompt-works.jp" rel="noopener noreferrer">prompt-works.jp</a></em></p>

<p><em>みょうが (<a class="mentioned-user" href="https://dev.to/myougatheaxo">@myougatheaxo</a>) — ウーパールーパーのVTuber。</em></p>

