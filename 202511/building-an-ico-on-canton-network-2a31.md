---
Title: Building an ICO on Canton Network
Description: 
Author: Dennison Bertram
Date: 2025-11-12T21:06:22.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>A hands-on guide to creating privacy-preserving token sales on Canton Network</strong></p>

<p>Ever wanted to build a token sale where only the buyer, seller, and maybe a regulator can see the details? That's exactly what we're going to create today. We'll build a complete ICO (Initial Coin Offering) smart contract that sells one token for another, with privacy built-in and atomic operations that can't fail halfway through.</p>




<h2>
  
  
  What's an ICO Anyway?
</h2>

<p>ICO's are back! From <a href="https://www.monad.xyz" rel="noopener noreferrer">Monad</a> to <a href="//megaeth.com">MegaETH</a> or projects launching their token on <a href="https://www.tally.xyz" rel="noopener noreferrer">Tally.xyz</a>, capital formation as product market fit for crypto is the meta.</p>

<p>Before we dive into code, let's make sure we're on the same page. An ICO (Initial Coin Offering) is when a project sells their tokens to raise funds. Traditionally this happens on public blockchains where everyone can see:</p>

<ul>
<li>Who bought what</li>
<li>How much they paid</li>
<li>All the transaction details</li>
</ul>

<p>But what if you want to keep those details private? That's where Canton Network comes in.</p>




<h2>
  
  
  Why Canton Network?
</h2>

<p><a href="https://www.canton.network/" rel="noopener noreferrer">Canton Network</a> is different. It gives you:</p>

<p>🔒 <strong>Privacy by Default</strong><br>
Only the parties directly involved in a transaction can see the details. No more public order books!</p>

<p>⚡ <strong>Atomic Operations</strong><br>
Either everything happens (buyer pays, seller delivers) or nothing happens. No failed transactions leaving things in a weird state.</p>

<p>🤝 <strong>Multi-Party Authorization</strong><br>
Smart contracts can require multiple parties to agree before anything happens. Perfect for regulated token sales.</p>

<p>🏗️ <strong>Enterprise Ready</strong><br>
Built for serious applications with proper tooling, monitoring, and compliance features.</p>


<h2>
  
  
  What We're Building
</h2>

<p>We'll create a complete ICO system with three parts:</p>

<ol>
<li>
<strong>Token Contract</strong> - A simple but secure token that can be issued and transferred</li>
<li>
<strong>ICO Contract</strong> - The main sale logic with atomic token-for-token swaps</li>
<li>
<strong>Test Suite</strong> - Scripts to verify everything works correctly</li>
</ol>

<p>The flow will be:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Buyer pays with USDC → ICO contract validates → MYCOIN tokens created → All in one atomic transaction
</code></pre>

</div>






<h2>
  
  
  Quick Setup
</h2>

<p>Let's get your environment ready. You'll need:</p>

<ul>
<li>
<strong>Daml SDK</strong> (version 3.3.0-snapshot...)</li>
<li>
<strong>Java 21</strong> (for the build tools)</li>
<li>
<strong>Docker</strong> (for running the local network)</li>
<li>
<strong>tmux</strong> (for managing long-running processes)</li>
</ul>

<p>The easiest way is to clone the Canton quickstart repo and run their setup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/digital-asset/cn-quickstart.git
<span class="nb">cd </span>cn-quickstart/quickstart
make setup
</code></pre>

</div>



<p>This gives you a complete development environment with Canton nodes, wallets, and monitoring tools.</p>

<p>Want to get started with AI? Use this <a href="https://gist.github.com/dennisonbertram/9e495d6114077ed7fefbaeb2178092a4" rel="noopener noreferrer">llms.txt</a> to get started fast. </p>




<h2>
  
  
  Your First ICO Contract
</h2>

<p>Let's start with a simple token contract. Create a file called <code>Token.daml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>module Token where

import DA.Assert

-- A simple token that can be issued and transferred
template Token
  with
    issuer : Party     -- Who created this token
    owner : Party      -- Who currently owns it
    symbol : Text      -- Like "USD" or "MYCOIN"
    amount : Decimal   -- How much
  where
    ensure amount &gt; 0.0
    signatory issuer, owner  -- Both must agree to create it

    -- Transfer ownership to someone else
    choice Transfer : ContractId Token
      with
        newOwner : Party
      controller owner, newOwner  -- Both current and new owner must agree
      do
        create this with owner = newOwner

-- The authority that can create new tokens
template TokenIssuer
  with
    issuer : Party
    symbol : Text
    totalSupply : Decimal
  where
    signatory issuer

    -- Create new tokens for someone (doesn't consume this contract)
    nonconsuming choice IssueTokens : ContractId Token
      with
        recipient : Party
        amount : Decimal
      controller issuer, recipient  -- Both issuer and recipient must agree
      do
        assert (amount &gt; 0.0)
        create Token with
          issuer = issuer
          owner = recipient
          symbol = symbol
          amount = amount
</code></pre>

</div>



<p>Now let's create the ICO contract in <code>Ico.daml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>module Ico where

import DA.Assert
import DA.Time
import Token

-- An active ICO where people can buy tokens
template IcoOffering
  with
    issuer : Party                    -- The company running the ICO
    saleTokenIssuer : Party           -- Who creates the tokens being sold
    saleTokenSymbol : Text            -- What token is being sold
    paymentTokenIssuer : Party        -- Who creates the payment tokens
    paymentTokenSymbol : Text         -- What token buyers pay with
    exchangeRate : Decimal            -- How many sale tokens per payment token
    totalSaleTokens : Decimal         -- Total available for sale
    tokensSold : Decimal              -- How many sold so far
    totalRaised : Decimal             -- Total payment collected
    startTime : Time                  -- When ICO starts
    endTime : Time                    -- When ICO ends
    minPurchase : Decimal             -- Minimum purchase amount
    maxPurchase : Decimal             -- Maximum purchase (0 = no limit)
  where
    ensure totalSaleTokens &gt; 0.0 &amp;&amp; exchangeRate &gt; 0.0
    signatory issuer

    -- Buy tokens! This is the magic atomic swap
    choice Purchase : (ContractId Token.Token, ContractId IcoOffering)
      with
        buyer : Party
        paymentTokenCid : ContractId Token.Token  -- Buyer's payment token
        paymentAmount : Decimal                   -- How much they're paying
      controller buyer, issuer, saleTokenIssuer   -- All three parties must agree!
      do
        -- Check if ICO is active
        now &lt;- getTime
        assert (now &gt;= startTime)
        assert (now &lt; endTime)

        -- Validate purchase amount
        assert (paymentAmount &gt;= minPurchase)
        assert (maxPurchase == 0.0 || paymentAmount &lt;= maxPurchase)

        -- Calculate how many tokens they get
        let saleTokenAmount = paymentAmount * exchangeRate

        -- Make sure we have enough tokens left
        let remainingTokens = totalSaleTokens - tokensSold
        assert (saleTokenAmount &lt;= remainingTokens)

        -- Verify they own the payment token
        paymentToken &lt;- fetch paymentTokenCid
        assert (paymentToken.issuer == paymentTokenIssuer)
        assert (paymentToken.symbol == paymentTokenSymbol)
        assert (paymentToken.owner == buyer)
        assert (paymentToken.amount &gt;= paymentAmount)

        -- Handle payment (split if needed, then transfer)
        if paymentToken.amount == paymentAmount
          then do
            -- Exact amount: transfer the whole token
            _ &lt;- exercise paymentTokenCid Token.Transfer with
                newOwner = issuer
            return ()
          else do
            -- Partial payment: split and transfer just the payment amount
            (paymentPortionCid, _changeCid) &lt;- exercise paymentTokenCid Token.Split with
                splitAmount = paymentAmount
            _ &lt;- exercise paymentPortionCid Token.Transfer with
                newOwner = issuer
            return ()

        -- Create the sale tokens for the buyer
        saleTokenCid &lt;- create Token.Token with
          issuer = saleTokenIssuer
          owner = buyer
          symbol = saleTokenSymbol
          amount = saleTokenAmount

        -- Update the ICO with new totals
        updatedIcoCid &lt;- create this with
          tokensSold = tokensSold + saleTokenAmount
          totalRaised = totalRaised + paymentAmount

        return (saleTokenCid, updatedIcoCid)

    -- Close the ICO when it's done
    choice Close : ContractId IcoCompleted
      controller issuer
      do
        now &lt;- getTime
        assert (now &gt;= endTime || tokensSold &gt;= totalSaleTokens)
        create IcoCompleted with
          issuer = issuer
          saleTokenIssuer = saleTokenIssuer
          saleTokenSymbol = saleTokenSymbol
          paymentTokenIssuer = paymentTokenIssuer
          paymentTokenSymbol = paymentTokenSymbol
          totalSaleTokens = totalSaleTokens
          tokensSold = tokensSold
          totalRaised = totalRaised
          finalExchangeRate = exchangeRate

-- Record of a completed ICO
template IcoCompleted
  with
    issuer : Party
    saleTokenIssuer : Party
    saleTokenSymbol : Text
    paymentTokenIssuer : Party
    paymentTokenSymbol : Text
    totalSaleTokens : Decimal
    tokensSold : Decimal
    totalRaised : Decimal
    finalExchangeRate : Decimal
  where
    signatory issuer

    -- Get final statistics
    nonconsuming choice GetStats : (Decimal, Decimal, Decimal)
      controller issuer
      do return (tokensSold, totalRaised, finalExchangeRate)

-- Helper to create ICOs
template IcoFactory
  with
    issuer : Party
  where
    signatory issuer

    choice CreateIco : ContractId IcoOffering
      with
        saleTokenIssuer : Party
        saleTokenSymbol : Text
        paymentTokenIssuer : Party
        paymentTokenSymbol : Text
        exchangeRate : Decimal
        totalSaleTokens : Decimal
        startTime : Time
        endTime : Time
        minPurchase : Decimal
        maxPurchase : Decimal
      controller issuer
      do
        assert (totalSaleTokens &gt; 0.0)
        assert (exchangeRate &gt; 0.0)
        assert (endTime &gt; startTime)
        assert (minPurchase &gt; 0.0)

        create IcoOffering with
          issuer = issuer
          saleTokenIssuer = saleTokenIssuer
          saleTokenSymbol = saleTokenSymbol
          paymentTokenIssuer = paymentTokenIssuer
          paymentTokenSymbol = paymentTokenSymbol
          exchangeRate = exchangeRate
          totalSaleTokens = totalSaleTokens
          tokensSold = 0.0
          totalRaised = 0.0
          startTime = startTime
          endTime = endTime
          minPurchase = minPurchase
          maxPurchase = maxPurchase
</code></pre>

</div>






<h2>
  
  
  Testing It Out
</h2>

<p>Let's create a test script to verify our ICO works. Create <code>IcoTest.daml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>module IcoTest where

import DA.Assert
import DA.Time
import Daml.Script
import Ico
import Token

test_ico_lifecycle = script do
  -- Create all the parties
  icoIssuer &lt;- allocateParty "Company"
  saleTokenIssuer &lt;- allocateParty "Company"
  paymentTokenIssuer &lt;- allocateParty "USDCIssuer"
  buyer1 &lt;- allocateParty "Alice"
  buyer2 &lt;- allocateParty "Bob"

  -- Create token issuers
  saleIssuerCid &lt;- submit saleTokenIssuer do
    createCmd TokenIssuer with
      issuer = saleTokenIssuer
      symbol = "MYCOIN"
      totalSupply = 1000000.0

  paymentIssuerCid &lt;- submit paymentTokenIssuer do
    createCmd TokenIssuer with
      issuer = paymentTokenIssuer
      symbol = "USDC"
      totalSupply = 1000000.0

  -- Give buyers some USDC to spend
  buyer1PaymentCid &lt;- submitMulti [paymentTokenIssuer, buyer1] [] do
    exerciseCmd paymentIssuerCid IssueTokens with
      recipient = buyer1
      amount = 1000.0

  buyer2PaymentCid &lt;- submitMulti [paymentTokenIssuer, buyer2] [] do
    exerciseCmd paymentIssuerCid IssueTokens with
      recipient = buyer2
      amount = 500.0

  -- Set up the ICO
  now &lt;- getTime
  factoryCid &lt;- submit icoIssuer do
    createCmd IcoFactory with issuer = icoIssuer

  icoCid &lt;- submit icoIssuer do
    exerciseCmd factoryCid CreateIco with
      saleTokenIssuer = saleTokenIssuer
      saleTokenSymbol = "MYCOIN"
      paymentTokenIssuer = paymentTokenIssuer
      paymentTokenSymbol = "USDC"
      exchangeRate = 100.0  -- 1 USDC = 100 MYCOIN
      totalSaleTokens = 50000.0
      startTime = now
      endTime = addRelTime now (days 1)
      minPurchase = 10.0
      maxPurchase = 0.0

  -- Alice buys 100 USDC worth (gets 10,000 MYCOIN)
  (aliceTokensCid, icoCid1) &lt;- submitMulti [buyer1, icoIssuer, saleTokenIssuer] [] do
    exerciseCmd icoCid Purchase with
      buyer = buyer1
      paymentTokenCid = buyer1PaymentCid
      paymentAmount = 100.0

  -- Verify Alice got her tokens
  Some aliceTokens &lt;- queryContractId buyer1 aliceTokensCid
  assert (aliceTokens.amount == 10000.0)
  assert (aliceTokens.symbol == "MYCOIN")

  -- Bob buys 50 USDC worth (gets 5,000 MYCOIN)
  (bobTokensCid, icoCid2) &lt;- submitMulti [buyer2, icoIssuer, saleTokenIssuer] [] do
    exerciseCmd icoCid1 Purchase with
      buyer = buyer2
      paymentTokenCid = buyer2PaymentCid
      paymentAmount = 50.0

  -- Verify Bob got his tokens
  Some bobTokens &lt;- queryContractId buyer2 bobTokensCid
  assert (bobTokens.amount == 5000.0)

  -- Check ICO totals
  Some finalIco &lt;- queryContractId icoIssuer icoCid2
  assert (finalIco.tokensSold == 15000.0)  -- 10k + 5k
  assert (finalIco.totalRaised == 150.0)   -- 100 + 50

  return ()
</code></pre>

</div>






<h2>
  
  
  Testing It Out
</h2>

<p>Run the tests to make sure everything works:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># In the cn-quickstart/quickstart directory</span>
<span class="nb">cd </span>daml/ico-token
daml <span class="nb">test</span>
</code></pre>

</div>



<p>You should see something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Test suite passed with 8 transactions and 9 active contracts
</code></pre>

</div>






<h2>
  
  
  Deploying to LocalNet
</h2>

<p>Time to see your ICO running on a real Canton network! The build scripts handle all the complexity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Build the contracts</span>
<span class="nb">cd </span>examples/ico-token
./scripts/build.sh

<span class="c"># Deploy to local network</span>
./scripts/deploy.sh
</code></pre>

</div>



<p>This will:</p>

<ol>
<li>Compile your Daml contracts into a DAR file</li>
<li>Start Canton participant nodes</li>
<li>Launch web interfaces for exploration</li>
<li>Deploy your contracts to the ledger</li>
</ol>

<p>Once running, you can access:</p>

<ul>
<li>
<strong>Scan UI</strong>: <a href="http://scan.localhost:4000" rel="noopener noreferrer">http://scan.localhost:4000</a> (explore contracts)</li>
<li>
<strong>Wallet UI</strong>: <a href="http://wallet.localhost:2000" rel="noopener noreferrer">http://wallet.localhost:2000</a> (manage tokens)</li>
<li>
<strong>App Provider</strong>: <a href="http://app-provider.localhost:3000" rel="noopener noreferrer">http://app-provider.localhost:3000</a> (frontend)</li>
</ul>




<h2>
  
  
  Making It Real
</h2>

<p>Your ICO is now live on LocalNet! Here's how to interact with it:</p>

<ol>
<li>
<strong>Create parties</strong> through the wallet interfaces</li>
<li>
<strong>Issue tokens</strong> using the TokenIssuer contracts</li>
<li>
<strong>Set up an ICO</strong> using the IcoFactory</li>
<li>
<strong>Make purchases</strong> through the Purchase choice</li>
<li>
<strong>Watch the magic</strong> happen atomically</li>
</ol>

<p>The best part? Only the buyer, seller, and sale token issuer can see the transaction details. Everyone else just sees that a transaction happened.</p>




<h2>
  
  
  What's Next?
</h2>

<p>🎉 <strong>Congratulations!</strong> You just built a privacy-preserving ICO on Canton Network.</p>

<p>Want to take it further? Here are some ideas:</p>

<h3>
  
  
  <strong>Add Features</strong>
</h3>

<ul>
<li>
<strong>Tiered pricing</strong>: Different rates for different purchase amounts</li>
<li>
<strong>Time bonuses</strong>: Better rates for early buyers</li>
<li>
<strong>Vesting</strong>: Lock purchased tokens for a period</li>
<li>
<strong>KYC integration</strong>: Verify buyer identities</li>
</ul>

<h3>
  
  
  <strong>Go Production</strong>
</h3>

<ul>
<li>
<strong>Multi-party deployments</strong>: Run across different Canton domains</li>
<li>
<strong>Compliance</strong>: Add regulator observers</li>
<li>
<strong>Monitoring</strong>: Set up proper observability</li>
<li>
<strong>Scaling</strong>: Handle thousands of concurrent buyers</li>
</ul>

<h3>
  
  
  <strong>Learn More</strong>
</h3>

<ul>
<li>
<strong>Daml patterns</strong>: Explore more Canton-specific smart contract patterns</li>
<li>
<strong>Privacy models</strong>: Deep dive into sub-transaction privacy</li>
<li>
<strong>Integration</strong>: Connect with existing DeFi protocols</li>
</ul>

<p>The Canton ecosystem is growing fast. Your ICO contract is a great foundation for building the next generation of token sales!</p>




<h2>
  
  
  About the Author
</h2>

<p>Dennison Bertram is the co-founder and CEO of <a href="https://www.tally.xyz" rel="noopener noreferrer">Tally.xyz</a>, the infrastructure layer for tokens—from ICO and airdrop to governance, staking, and value accrual. Tally delivers the complete framework for operating tokens at scale, powering the largest teams in the ecosystem with billions under management.</p>

