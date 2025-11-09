---
Title: Building CanopySplit: A Deep Dive into Octant V2 Yield Donating Strategies
Description: 
Author: N DIVIJ
Date: 2025-11-09T22:02:34.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p><strong>CanopySplit</strong> is a multi-layer DeFi protocol that transforms idle capital into climate impact. Users deposit WETH, earn yield through Aave v3, and automatically split 100% of profits among climate recipients—all while keeping their principal withdrawable at any time.</p>

<p>This technical deep-dive covers:</p>

<ul>
<li>Octant V2 Yield Donating Strategy architecture</li>
<li>ERC-4626 vault integration with Aave v3</li>
<li>Epoch-based donation splitting mechanics</li>
<li>Custom Uniswap v4 hook for swap fee donations</li>
<li>Production deployment challenges on Sepolia</li>
</ul>

<p><strong>Live on Sepolia:</strong></p>

<ul>
<li>Strategy: <code>0x0D1d8AE2dD0e4B06ca0Ef2949150eb021cAf6Ce9</code>
</li>
<li>Splitter: <code>0xda5fA1c26Ec29497C2B103B385569286B30EC248</code>
</li>
<li>Asset (WETH): <code>0xC558DBdd856501FCd9aaF1E62eae57A9F0629a3c</code>
</li>
</ul>




<h2>
  
  
  Architecture Overview
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────┐
│ User Wallet │
└──────┬──────┘
       │ deposit WETH
       ▼
┌──────────────────────────────┐
│ Aave4626YieldDonatingStrategy│ (ERC-4626 vault)
│ - Deposits to Aave v3        │
│ - Mints shares to users      │
│ - Reports profit → splitter  │
└──────────────┬───────────────┘
               │ mint donation shares
               ▼
┌──────────────────────────────┐
│ TriSplitDonationSplitter     │
│ - Holds strategy shares      │
│ - Epoch-based allocation     │
│ - 3 climate recipients       │
└──────┬───────────────────────┘
       │ distribute()
       ▼
┌─────────────────────────────────┐
│ Recipients (50% / 30% / 20%)    │
│ - Planters                      │
│ - MRV (Monitoring/Verification) │
│ - Maintenance                   │
└─────────────────────────────────┘
</code></pre>

</div>






<h2>
  
  
  Part 1: Octant V2 Yield Donating Strategy
</h2>

<h3>
  
  
  The Core Concept
</h3>

<p>Octant V2 introduces <strong>Yield Donating Strategies</strong> (YDS), which extend ERC-4626 with donation mechanics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>interface IYieldDonatingStrategy is IERC4626 {
    function report() external returns (uint256 profit, uint256 loss);
    function setDonationSplitter(address splitter) external;
}
</code></pre>

</div>



<p><strong>Key innovation:</strong> When <code>report()</code> is called:</p>

<ol>
<li>Calculate <code>profit = currentAssets - lastRecordedAssets</code>
</li>
<li>If profit &gt; 0: <strong>Mint shares to splitter</strong> (not to users!)</li>
<li>If loss &gt; 0: <strong>Burn donation shares first</strong>, protecting user capital</li>
</ol>

<p>This means:</p>

<ul>
<li>Users keep 100% of principal</li>
<li>Donations absorb losses first</li>
<li>No performance fees</li>
</ul>

<h3>
  
  
  Implementation: Aave4626YieldDonatingStrategy
</h3>

<p>We built an ERC-4626 wrapper around Aave v3:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>contract Aave4626YieldDonatingStrategy is YieldDonatingTokenizedStrategy {
    IERC4626 public immutable vault; // ATokenVault

    constructor(
        IERC20 asset_,
        IERC4626 vault_,
        string memory name_,
        address management_,
        address keeper_
    ) YieldDonatingTokenizedStrategy(asset_, name_, management_, keeper_) {
        vault = vault_;
        asset_.forceApprove(address(vault_), type(uint256).max);
    }

    function totalAssets() public view override returns (uint256) {
        uint256 shares = vault.balanceOf(address(this));
        return vault.convertToAssets(shares);
    }

    function _deployFunds(uint256 amount) internal override {
        vault.deposit(amount, address(this));
    }

    function _freeFunds(uint256 amount) internal override {
        vault.withdraw(amount, address(this), address(this));
    }
}
</code></pre>

</div>



<p><strong>Why ERC-4626?</strong></p>

<ul>
<li>
<strong>Composability:</strong> Any ERC-4626 vault can plug in</li>
<li>
<strong>Non-rebasing:</strong> Fixed shares, yield in <code>convertToAssets()</code>
</li>
<li>
<strong>Safety:</strong> Vault enforces supply caps and liquidity checks</li>
</ul>

<h3>
  
  
  The report() Flow
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function report() external onlyKeepers returns (uint256 profit, uint256 loss) {
    uint256 currentAssets = totalAssets();
    uint256 lastAssets = lastRecordedAssets;

    if (currentAssets &gt; lastAssets) {
        profit = currentAssets - lastAssets;

        // Mint shares to splitter (donation\!)
        uint256 sharesToMint = convertToShares(profit);
        _mint(donationSplitter, sharesToMint);

        emit Reported(profit, 0);
    } else if (currentAssets &lt; lastAssets) {
        loss = lastAssets - currentAssets;

        // Burn donation shares first
        uint256 splitterShares = balanceOf(donationSplitter);
        uint256 splitterAssets = convertToAssets(splitterShares);

        if (splitterAssets &gt;= loss) {
            uint256 sharesToBurn = convertToShares(loss);
            _burn(donationSplitter, sharesToBurn);
        } else {
            _burn(donationSplitter, splitterShares);
            // Remaining loss socialized across all shares
        }

        emit Reported(0, loss);
    }

    lastRecordedAssets = totalAssets();
}
</code></pre>

</div>



<p><strong>Critical insight:</strong> Minting shares (instead of transferring assets) means donations continue earning yield until distributed.</p>




<h2>
  
  
  Part 2: Aave v3 Integration
</h2>

<h3>
  
  
  ATokenVault: ERC-4626 Wrapper for Aave
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>contract ATokenVault is ERC4626, Ownable {
    IPool public immutable aavePool;
    IERC20 public immutable aToken;

    constructor(
        IERC20 asset_,
        IPoolAddressesProvider provider
    ) ERC4626(asset_) ERC20("Aave WETH Vault", "aWETHv") {
        aavePool = IPool(provider.getPool());
        aToken = IERC20(aavePool.getReserveData(address(asset_)).aTokenAddress);
    }

    function totalAssets() public view override returns (uint256) {
        return aToken.balanceOf(address(this)); // aTokens auto-accrue
    }

    function _deposit(address caller, address receiver, uint256 assets, uint256 shares) 
        internal override 
    {
        SafeERC20.safeTransferFrom(asset(), caller, address(this), assets);
        SafeERC20.forceApprove(asset(), address(aavePool), assets);

        aavePool.supply(address(asset()), assets, address(this), 0);

        _mint(receiver, shares);
        emit Deposit(caller, receiver, assets, shares);
    }

    function _withdraw(address caller, address receiver, address owner, uint256 assets, uint256 shares) 
        internal override 
    {
        if (caller \!= owner) {
            _spendAllowance(owner, caller, shares);
        }

        _burn(owner, shares);
        aavePool.withdraw(address(asset()), assets, receiver);

        emit Withdraw(caller, receiver, owner, assets, shares);
    }
}
</code></pre>

</div>



<p><strong>Deployed on Sepolia:</strong></p>

<ul>
<li>Vault: <code>0x6938238e57CBe1b4B51Eb3B51389cEf8d3a88521</code>
</li>
<li>Asset: WETH (<code>0xC558DBdd856501FCd9aaF1E62eae57A9F0629a3c</code>)</li>
<li>Aave Provider: <code>0x012bAC54348C0E635dCAc9D5FB99f06F24136C9A</code>
</li>
</ul>




<h2>
  
  
  Part 3: TriSplit Donation Splitter
</h2>

<h3>
  
  
  Epoch-Based Allocation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>struct Recipient {
    address addr;
    string role;
}

struct Policy {
    uint256 epoch;
    uint256[3] basisPoints; // [5000, 3000, 2000] = 50%, 30%, 20%
}

contract TriSplitDonationSplitter {
    Recipient[3] public recipients;
    Policy public currentPolicy;
    Policy public upcomingPolicy;
    IERC4626 public vault;

    function distribute() external {
        uint256 shares = vault.balanceOf(address(this));
        uint256 assets = vault.redeem(shares, address(this), address(this));

        uint256[3] memory amounts;
        for (uint256 i = 0; i &lt; 3; i++) {
            amounts[i] = (assets * currentPolicy.basisPoints[i]) / 10_000;
            IERC20(vault.asset()).safeTransfer(recipients[i].addr, amounts[i]);
        }

        emit Distributed(currentPolicy.epoch, amounts, recipients);
    }

    function rollEpoch() external onlyOwner {
        currentPolicy = upcomingPolicy;
        upcomingPolicy.epoch++;
        emit EpochRolled(currentPolicy.epoch);
    }

    function setUpcomingPolicy(uint256[3] calldata bps) external onlyOwner {
        require(bps[0] + bps[1] + bps[2] == 10_000, "Must sum to 100%");
        upcomingPolicy.basisPoints = bps;
        emit UpcomingPolicySet(upcomingPolicy.epoch, bps);
    }
}
</code></pre>

</div>



<p><strong>Why epochs?</strong></p>

<ul>
<li>Recipients know their allocation in advance</li>
<li>Owner can adjust future weights without disrupting current distributions</li>
<li>Full on-chain transparency via events</li>
</ul>

<p><strong>Current allocation:</strong></p>

<ul>
<li>Planters: 50% (<code>0xF9b2eFCAcc1B93c1bd7F898d0a8c4b34aBD78E53</code>)</li>
<li>MRV: 30% (<code>0x9261432cab3c0F83E86fa6e41E4a88dA06E7ecc6</code>)</li>
<li>Maintenance: 20% (<code>0x89C13e8e5a81E775160322df9d7869893926A8Cc</code>)</li>
</ul>




<h2>
  
  
  Part 4: Uniswap v4 Hook (Local PoC)
</h2>

<h3>
  
  
  Extending Donations to Swap Fees
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>contract TriSplitDonationHook is BaseHook {
    struct HookConfig {
        address donationTarget;
        uint256 donationBps; // 100 = 1%
    }

    mapping(PoolId =&gt; HookConfig) public configs;

    function afterSwap(
        address,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        BalanceDelta delta,
        bytes calldata
    ) external override returns (bytes4, int128) {
        PoolId poolId = key.toId();
        HookConfig memory config = configs[poolId];

        if (config.donationTarget \!= address(0)) {
            Currency currency = params.zeroForOne ? key.currency0 : key.currency1;

            uint256 amountIn = params.zeroForOne 
                ? uint256(int256(-delta.amount0())) 
                : uint256(int256(-delta.amount1()));

            uint256 donation = (amountIn * config.donationBps) / 10_000;

            poolManager.take(currency, config.donationTarget, donation);

            emit DonationExecuted(poolId, donation, config.donationTarget);
        }

        return (this.afterSwap.selector, 0);
    }
}
</code></pre>

</div>



<p><strong>Testing on Anvil:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Deploy v4 infrastructure</span>
forge script script/00_DeployV4.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545

<span class="c"># Deploy hook</span>
forge script script/00_DeployHook.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545

<span class="c"># Initialize pool</span>
forge script script/05_EnsureInitialized.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545

<span class="c"># Add liquidity</span>
forge script script/02_AddLiquidity.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545

<span class="c"># Configure hook (1% donation)</span>
forge script script/04_SetHookConfig.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545

<span class="c"># Execute swap (triggers donation\!)</span>
forge script script/03_Swap.s.sol <span class="nt">--broadcast</span> <span class="nt">--rpc-url</span> http://127.0.0.1:8545
</code></pre>

</div>






<h2>
  
  
  Part 5: Frontend Architecture
</h2>

<h3>
  
  
  Dynamic Asset Detection
</h3>

<p>Instead of hardcoding USDC/WETH, we read the asset dynamically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">useAssetInfo</span><span class="p">(</span><span class="nx">strategyAddress</span><span class="p">:</span> <span class="nx">Address</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">assetAddress</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useReadContract</span><span class="p">({</span>
    <span class="na">address</span><span class="p">:</span> <span class="nx">strategyAddress</span><span class="p">,</span>
    <span class="na">abi</span><span class="p">:</span> <span class="nx">StrategyABI</span><span class="p">,</span>
    <span class="na">functionName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">asset</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">symbol</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useReadContract</span><span class="p">({</span>
    <span class="na">address</span><span class="p">:</span> <span class="nx">assetAddress</span><span class="p">,</span>
    <span class="na">abi</span><span class="p">:</span> <span class="nx">ERC20ABI</span><span class="p">,</span>
    <span class="na">functionName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">symbol</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">decimals</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useReadContract</span><span class="p">({</span>
    <span class="na">address</span><span class="p">:</span> <span class="nx">assetAddress</span><span class="p">,</span>
    <span class="na">abi</span><span class="p">:</span> <span class="nx">ERC20ABI</span><span class="p">,</span>
    <span class="na">functionName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">decimals</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">assetAddress</span><span class="p">,</span> <span class="nx">symbol</span><span class="p">,</span> <span class="nx">decimals</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Benefits:</strong></p>

<ul>
<li>Works with any ERC-20 asset</li>
<li>No frontend redeployment needed</li>
<li>Correct decimal formatting</li>
</ul>

<h3>
  
  
  Role-Based Access Control
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">AppPage</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">address</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAccount</span><span class="p">();</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">management</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useReadContract</span><span class="p">({</span>
    <span class="na">address</span><span class="p">:</span> <span class="nx">ADDRS</span><span class="p">.</span><span class="nx">sepolia</span><span class="p">.</span><span class="nx">strategy</span><span class="p">,</span>
    <span class="na">abi</span><span class="p">:</span> <span class="nx">StrategyABI</span><span class="p">,</span>
    <span class="na">functionName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">management</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">isManagement</span> <span class="o">=</span> <span class="nx">address</span> <span class="o">===</span> <span class="nx">management</span><span class="p">;</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;&gt;</span>
      <span class="o">&lt;</span><span class="nx">DepositForm</span> <span class="o">/&gt;</span>
      <span class="o">&lt;</span><span class="nx">WithdrawForm</span> <span class="o">/&gt;</span>

      <span class="p">{</span><span class="nx">isManagement</span> <span class="o">&amp;&amp;</span> <span class="p">(</span>
        <span class="o">&lt;</span><span class="nx">Button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{</span><span class="nx">handleReport</span><span class="p">}</span><span class="o">&gt;</span><span class="nx">Report</span> <span class="nx">Profit</span><span class="o">/</span><span class="nx">Loss</span><span class="o">&lt;</span><span class="sr">/Button</span><span class="err">&gt;
</span>      <span class="p">)}</span>
    <span class="o">&lt;</span><span class="sr">/</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Event Tracking
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">useLifetimeDonations</span><span class="p">(</span><span class="nx">splitterAddress</span><span class="p">:</span> <span class="nx">Address</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">logs</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useContractEvent</span><span class="p">({</span>
    <span class="na">address</span><span class="p">:</span> <span class="nx">splitterAddress</span><span class="p">,</span>
    <span class="na">abi</span><span class="p">:</span> <span class="nx">SplitterABI</span><span class="p">,</span>
    <span class="na">eventName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Distributed</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">fromBlock</span><span class="p">:</span> <span class="mi">0</span><span class="nx">n</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">totalDonated</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="err">\</span><span class="o">!</span><span class="nx">logs</span><span class="p">)</span> <span class="k">return</span> <span class="mi">0</span><span class="nx">n</span><span class="p">;</span>
    <span class="k">return</span> <span class="nx">logs</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">sum</span><span class="p">,</span> <span class="nx">log</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="p">[</span><span class="nx">epoch</span><span class="p">,</span> <span class="nx">amounts</span><span class="p">]</span> <span class="o">=</span> <span class="nx">log</span><span class="p">.</span><span class="nx">args</span><span class="p">;</span>
      <span class="k">return</span> <span class="nx">sum</span> <span class="o">+</span> <span class="nx">amounts</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">,</span> <span class="mi">0</span><span class="nx">n</span><span class="p">);</span>
    <span class="p">},</span> <span class="mi">0</span><span class="nx">n</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">logs</span><span class="p">]);</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">totalDonated</span><span class="p">,</span> <span class="na">eventCount</span><span class="p">:</span> <span class="nx">logs</span><span class="p">?.</span><span class="nx">length</span> <span class="o">??</span> <span class="mi">0</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Real-World Challenges
</h2>

<h3>
  
  
  Challenge 1: Sepolia Gas Cap (16.7M)
</h3>

<p><strong>Problem:</strong> Contract creation bytecode exceeded Sepolia's per-tx gas limit.</p>

<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># foundry.toml</span>
<span class="py">optimizer</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">optimizer_runs</span> <span class="p">=</span> <span class="mi">1</span>
<span class="py">via_ir</span> <span class="p">=</span> <span class="kc">false</span>
<span class="py">bytecode_hash</span> <span class="p">=</span> <span class="s">"none"</span>
<span class="py">cbor_metadata</span> <span class="p">=</span> <span class="kc">false</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>forge clean <span class="o">&amp;&amp;</span> forge build
forge create YieldDonatingTokenizedStrategy <span class="nt">--gas-limit</span> 0xFD0000 <span class="nt">--legacy</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 2: WETH Pivot
</h3>

<p><strong>Problem:</strong> Aave v3 Sepolia USDC had supply cap issues.</p>

<p><strong>Solution:</strong> Switched to WETH (18 decimals, no supply cap).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">USDC_UNDERLYING</span><span class="o">=</span>0xC558DBdd856501FCd9aaF1E62eae57A9F0629a3c  <span class="c"># WETH\!</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 3: Permit2 Allowance
</h3>

<p><strong>Problem:</strong> Swaps reverted with <code>InsufficientAllowance</code>.</p>

<p><strong>Root cause:</strong> Granted allowance to PoolManager, but Router calls <code>transferFrom()</code>.</p>

<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Grant allowance to Router, not PoolManager
permit2.approve(address(token0), address(swapRouter), type(uint160).max, type(uint48).max);
</code></pre>

</div>



<h3>
  
  
  Challenge 4: Token Decimals
</h3>

<p><strong>Problem:</strong> USDC (6 decimals) vs WETH (18 decimals) mismatches.</p>

<p><strong>Solution:</strong> Created patching script for local testing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FakeUSDC fake = new FakeUSDC(); // decimals() = 6
vm.etch(token0Addr, address(fake).code);
</code></pre>

</div>






<h2>
  
  
  Key Innovations
</h2>

<ol>
<li>
<strong>Epoch-Based Splits:</strong> Unlike single-recipient YDS, we route to 3 recipients with dynamic weights</li>
<li>
<strong>Multi-Adapter Support:</strong> Idle, Aave v3, and ERC-4626 strategies all feed the same splitter</li>
<li>
<strong>Dynamic Asset Detection:</strong> Frontend works with any ERC-20 without hardcoding</li>
<li>
<strong>Hooks Integration:</strong> Extended YDS concept to Uniswap v4 swap fees</li>
<li>
<strong>Loss Protection:</strong> Donation shares burn first, protecting user capital</li>
</ol>




<h2>
  
  
  Conclusion
</h2>

<p>CanopySplit demonstrates how Octant V2's Yield Donating Strategies can be extended with:</p>

<ul>
<li>
<strong>Multi-protocol integrations</strong> (Aave v3, Uniswap v4)</li>
<li>
<strong>Flexible donation mechanics</strong> (epoch-based splits, loss protection)</li>
<li>
<strong>Production-ready UX</strong> (dynamic asset detection, role-based access)</li>
</ul>

<p>The architecture is composable, transparent, and gas-efficient. All code is open-source and ready for judges to test.</p>

<p><strong>Key takeaways for builders:</strong></p>

<ol>
<li>ERC-4626 is the perfect abstraction for yield strategies</li>
<li>Epoch-based policies provide flexibility without disrupting current operations</li>
<li>Dynamic asset detection makes frontends future-proof</li>
<li>Testnet gas limits are real—optimize aggressively</li>
<li>Always verify contract addresses exist on-chain before trusting deployment logs</li>
</ol>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/N-45div/CanopySplit" rel="noopener noreferrer">https://github.com/N-45div/CanopySplit</a>
</li>
<li>
<strong>Sepolia Strategy:</strong> <code>0x0D1d8AE2dD0e4B06ca0Ef2949150eb021cAf6Ce9</code>
</li>
<li>
<strong>Sepolia Splitter:</strong> <code>0xda5fA1c26Ec29497C2B103B385569286B30EC248</code>
</li>
<li>
<strong>Octant V2 Docs:</strong> <a href="https://docs.octant.app" rel="noopener noreferrer">docs.octant.app</a>
</li>
</ul>




<p><em>Built with ❤️ for the Octant V2 hackathon. Questions? Reach out on Twitter <a href="https://twitter.com/godlovesu_n" rel="noopener noreferrer">@godlovesu_n</a></em></p>

