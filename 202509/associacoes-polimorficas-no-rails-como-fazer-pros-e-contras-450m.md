---
Title: Associações polimórficas no Rails: como fazer, prós e contras
Description: 
Author: Pedro Leonardo
Date: 2025-09-16T21:54:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Recentemente, me deparei com uma situação no trabalho em que precisava implementar uma tabela que se relacionasse com <strong>várias outras tabelas diferentes</strong>, mas que possuía os mesmos campos em comum. Pesquisando sobre a melhor forma de lidar com isso no <strong>Rails</strong>, descobri as <a href="https://guides.rubyonrails.org/association_basics.html#polymorphic-associations" rel="noopener noreferrer">associações polimórficas</a></p>

<p>Neste artigo, vamos aprender na prática como implementá-las e discutir suas <strong>vantagens, desvantagens e quando usar</strong>.</p>

<h2>
  
  
  O que são associações polimórficas?
</h2>

<p>Uma <strong>associação polimórfica</strong> é um recurso do Rails que permite que um mesmo modelo pertença a mais de um outro modelo, usando uma estrutura genérica.</p>

<p>Por exemplo, imagine que você tenha um sistema em que <strong>usuários e empresas</strong> podem ter <strong>contatos</strong> (e-mail, telefone). Em vez de criar duas tabelas (<code>user_contacts</code> e <code>company_contacts</code>), você pode ter uma tabela única de <code>contacts</code> que se relaciona com ambos.</p>

<h2>
  
  
  Criando a associação polimórfica
</h2>

<p>Implementar uma associação polimórfica no Rails é simples. Vamos criar uma tabela <code>contacts</code> que poderá se relacionar tanto com <code>users</code> quanto com <code>companies</code> (ou qualquer outro modelo que precise de contatos).</p>

<h3>
  
  
  Migration
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class CreateContacts &lt; ActiveRecord::Migration[7.1]
  def change
    create_table :contacts do |t|
      t.string :email
      t.string :phone

      t.references :contactable, polymorphic: true, null: false

      t.timestamps
    end
  end
end
</code></pre>

</div>



<blockquote>
<p>O <code>t.references :contactable, polymorphic: true</code> cria duas colunas:</p>

<ul>
<li>
<code>contactable_id</code> (inteiro)</li>
<li>
<code>contactable_type</code> (string)</li>
</ul>
</blockquote>

<p>Esses dois campos indicam quem é o dono do contato.</p>

<h3>
  
  
  Models
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Contact &lt; ApplicationRecord
  belongs_to :contactable, polymorphic: true
end

class User &lt; ApplicationRecord
  has_many :contacts, as: :contactable
end

class Company &lt; ApplicationRecord
  has_many :contacts, as: :contactable
end
</code></pre>

</div>



<p>Agora, tanto <code>User</code> quanto <code>Company</code> podem ter múltiplos contatos.</p>

<h3>
  
  
  Exemplo de uso
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>user = User.create(name: "Pedro")
company = Company.create(name: "ABC")

user.contacts.create(email: "pedro@email.com", phone: "1111-1111")
company.contacts.create(email: "contato@tech.com", phone: "2222-2222")

puts user.contacts.first.email     # =&gt; "pedro@email.com"
puts company.contacts.first.phone  # =&gt; "2222-2222"

</code></pre>

</div>



<p>Com isso, conseguimos reutilizar a tabela <code>contacts</code> em diferentes contextos.</p>

<h2>
  
  
  Vantagens
</h2>

<p><strong>Reutilização</strong>: você cria <strong>uma única tabela</strong> para relacionar com vários modelos diferentes.</p>

<p><strong>Menos duplicação</strong>: evita criar várias tabelas para armazenar dados semelhantes.</p>

<p><strong>Flexibilidade</strong>: se amanhã surgir outro modelo que precise de contatos (ex: <code>Suppliers</code>), basta adicionar a associação.</p>

<p><strong>Integração Rails</strong>: o suporte nativo facilita bastante, sem precisar reinventar a roda.</p>

<h2>
  
  
  Desvantagens
</h2>

<p><strong>Consultas mais complexas</strong>: filtrar dados polimórficos pode gerar queries mais pesadas.</p>

<p><strong>Dificuldade em manter integridade</strong>: o Rails não cria <em>foreign keys</em> em colunas polimórficas (já que o campo pode apontar para várias tabelas diferentes).</p>

<p><strong>Escalabilidade</strong>: se a tabela crescer demais com dados de muitos modelos, pode virar um gargalo.</p>

<p><strong>Legibilidade</strong>: para quem não conhece o conceito, entender o <code>contactable_type</code> pode ser confuso.</p>

<h2>
  
  
  Quando usar?
</h2>

<p><strong>Use quando</strong>:</p>

<ul>
<li>Você tem dados <strong>idênticos</strong> que precisam estar disponíveis em <strong>múltiplos modelos</strong> (ex: contatos, endereços, imagens, comentários).</li>
<li>A estrutura se repete bastante e faria pouco sentido criar tabelas separadas.</li>
</ul>

<p><strong>Evite quando</strong>:</p>

<ul>
<li>Os dados variam muito entre os modelos (ex: se contatos de <code>User</code> e <code>Company</code> tivessem campos completamente diferentes).</li>
<li>Você precisa de <strong>restrições fortes no banco</strong> (integridade referencial por foreign keys diretas).</li>
<li>A tabela polimórfica tende a crescer demais, impactando performance.</li>
</ul>

<h2>
  
  
  Alternativa às associações polimórficas
</h2>

<p>Uma alternativa às associações polimórficas é criar os relacionamentos de forma <strong>explícita</strong> na tabela, adicionando colunas específicas como <code>user_id</code>, <code>company_id</code>, <code>supplier_id</code>, etc.</p>

<h3>
  
  
  Exemplo de migration sem polimorfismo:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class CreateContacts &lt; ActiveRecord::Migration[7.1]
  def change
    create_table :contacts do |t|
      t.string :email
      t.string :phone

      # Relacionamentos explícitos
      t.references :user, foreign_key: true
      t.references :company, foreign_key: true

      t.timestamps
    end
  end
end
</code></pre>

</div>



<h3>
  
  
  Prós dessa abordagem:
</h3>

<ul>
<li>
<strong>Performance melhor</strong>: consultas são mais rápidas porque não dependem de um campo <code>type</code>.</li>
<li>
<strong>Integridade no banco</strong>: você consegue usar <strong>foreign keys</strong> diretamente, garantindo consistência.</li>
<li>
<strong>Legibilidade</strong>: é mais fácil entender a relação olhando apenas a tabela.</li>
</ul>

<h3>
  
  
  Contras dessa abordagem:
</h3>

<ul>
<li>
<strong>Pouca escalabilidade</strong>: se amanhã surgir outro modelo (ex: <code>Supplier</code>), você precisa <strong>alterar a tabela</strong> e adicionar mais colunas.</li>
<li>
<strong>Duplicação estrutural</strong>: quanto mais modelos diferentes, mais colunas redundantes terá na tabela.</li>
</ul>

<p>Essa opção costuma ser escolhida em cenários onde <strong>performance é prioridade</strong> e o número de modelos relacionados é previsível e limitado.</p>

<h2>
  
  
  Conclusão
</h2>

<p>Associações polimórficas são uma ferramenta poderosa no Rails. Elas trazem simplicidade e reuso quando bem aplicadas, mas podem gerar <strong>complexidade desnecessária</strong> e até <strong>impactar performance</strong> em cenários de alto volume de dados.</p>

<p>Uma alternativa válida é criar <strong>referências explícitas</strong> (<code>user_id</code>, <code>company_id</code>, etc.) em vez de usar o campo polimórfico. Essa abordagem melhora a performance e permite integridade via <strong>foreign keys</strong>, mas perde em <strong>flexibilidade</strong> e exige alterações na tabela sempre que surgir um novo modelo relacionado.</p>

<p>No fim, não existe solução única:</p>

<ul>
<li>Se você precisa de <strong>flexibilidade e reuso</strong> → vá de <strong>associação polimórfica</strong>.</li>
<li>Se você precisa de <strong>performance e integridade forte no banco</strong> → prefira <strong>chaves explícitas (<code>_id</code>)</strong>.</li>
</ul>

<p>A escolha depende do <strong>contexto e das prioridades do projeto</strong>.</p>

