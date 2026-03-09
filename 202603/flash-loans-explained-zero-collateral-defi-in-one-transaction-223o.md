---
Title: Flash Loans Explained: Zero-Collateral DeFi in One Transaction
Description: 
Author: metadevdigital
Date: 2026-03-09T22:11:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Flash Loans Explained: Zero-Collateral DeFi in One Transaction
</h1>

<p>I remember the first time I saw a flash loan attack in the wild. It was 2020, watching $45 million evaporate from bZx in a single block. My initial reaction? "That's... actually genius. And completely terrifying."</p>

<p>Flash loans broke something fundamental we thought we understood about finance: the collateral model. For years, everyone assumed DeFi needed overcollateralization because blockchains can't do credit checks. Then Aave said, "what if we just... don't?" You borrow millions, execute whatever trades you want, then repay plus fees—all in one atomic transaction. If you can't repay? The entire transaction reverts like it never happened.</p>

<p>This isn't theoretical anymore. Flash loans have become actual infrastructure. Legitimate arbitrage, liquidations, oracle fixes—they're everywhere now. But I've also watched them used for some of the slickest hacks imaginable. Understanding how they work isn't optional if you're building on-chain.</p>

<p>Let me walk you through the mechanics, show you the code, and tell you about the landmines I've stepped on.</p>

<h2>
  
  
  How Flash Loans Actually Work
</h2>

<p>The mechanic is deceptively simple. Aave (and now many others) lets you do this:</p>

<ol>
<li>Transfer tokens to you</li>
<li>Call a function you control</li>
<li>Wait for you to complete whatever logic you need</li>
<li>Check your balance before and after</li>
<li>If you repaid with fees, commit. If not, revert everything.</li>
</ol>

<p>It's like Aave is saying, "I trust the laws of physics more than I trust you as a person."<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Simplified Aave V3 flash loan interface
interface IFlashLoanReceiver {
    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external returns (bytes32);
}

contract SimpleFlashBorrower is IFlashLoanReceiver {
    ILendingPool public lendingPool;

    constructor(address _lendingPool) {
        lendingPool = ILendingPool(_lendingPool);
    }

    function executeFlashLoan(address asset, uint256 amount) external {
        lendingPool.flashLoan(
            address(this),
            asset,
            amount,
            abi.encode()
        );
    }

    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external override returns (bytes32) {
        // Execute your arbitrage, liquidation, swap sequence, whatever
        _doSomethingProfitable(asset, amount);

        // You MUST repay amount + premium before the function exits
        uint256 amountOwed = amount + premium;
        IERC20(asset).approve(address(lendingPool), amountOwed);

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    function _doSomethingProfitable(address asset, uint256 amount) internal {
        // Swap on Uniswap V3, arbitrage between exchanges, liquidate positions, etc.
    }
}
</code></pre>

</div>



<p>Here's the critical part: <strong>the entire transaction reverts if you can't repay</strong>. No loan hanging over your head. No liquidation risk. Just atomic execution.</p>

<h2>
  
  
  Real-World Example: Arbitrage Between DEXes
</h2>

<p>This finds price mismatches and exploits them. I actually tested this one:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pragma solidity ^0.8.0;

interface IUniswapV2Router {
    function swapExactTokensForTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
}

interface IUniswapV3SwapRouter {
    struct ExactInputParams {
        bytes path;
        address recipient;
        uint256 deadline;
        uint256 amountIn;
        uint256 amountOutMinimum;
    }

    function exactInput(ExactInputParams calldata params) 
        external returns (uint256 amountOut);
}

contract ArbitrageFlashLoan is IFlashLoanReceiver {
    ILendingPool public lendingPool;
    IUniswapV2Router public uniV2Router = 
        IUniswapV2Router(0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D); // Mainnet
    IUniswapV3SwapRouter public uniV3Router =
        IUniswapV3SwapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564); // Mainnet

    address constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
    address constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    constructor(address _lendingPool) {
        lendingPool = ILendingPool(_lendingPool);
    }

    function initiateArbitrage(uint256 usdcAmount) external {
        lendingPool.flashLoan(
            address(this),
            USDC,
            usdcAmount,
            abi.encode(usdcAmount)
        );
    }

    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external override returns (bytes32) {
        require(msg.sender == address(lendingPool), "Unauthorized");
        require(asset == USDC, "Wrong asset");

        uint256 borrowAmount = amount;
        uint256 fee = premium;

        // Swap USDC → WETH on Uniswap V2 (cheaper here)
        address[] memory pathV2 = new address[](2);
        pathV2[0] = USDC;
        pathV2[1] = WETH;

        IERC20(USDC).approve(address(uniV2Router), borrowAmount);
        uint[] memory amountsV2 = uniV2Router.swapExactTokensForTokens(
            borrowAmount,
            0,
            pathV2,
            address(this),
            block.timestamp + 300
        );

        uint256 wethReceived = amountsV2[amountsV2.length - 1];

        // Swap WETH → USDC on Uniswap V3 (more expensive, where we profit)
        IUniswapV3SwapRouter.ExactInputParams memory params3 = 
            IUniswapV3SwapRouter.ExactInputParams({
                path: abi.encodePacked(WETH, uint24(3000), USDC),
                recipient: address(this),
                deadline: block.timestamp + 300,
                amountIn: wethReceived,
                amountOutMinimum: borrowAmount + fee
            });

        IERC20(WETH).approve(address(uniV3Router), wethReceived);
        uniV3Router.exactInput(params3);

        // Repay the loan with premium
        uint256 amountOwed = borrowAmount + fee;
        IERC20(USDC).approve(address(lendingPool), amountOwed);

        // Your profit is whatever USDC is left

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }
}
</code></pre>

</div>



<p>In theory it's clean. In practice, I've seen this fail in about fifty different ways.</p>

<h2>
  
  
  The Gotchas I've Seen Burn People
</h2>

<p><strong>1. Price Slippage and the Vanishing Profit</strong></p>

<p>You calculate an arbitrage opportunity off-chain. By the time your transaction lands, mempool activity has shifted prices. Your "profitable" trade becomes a loss. Transaction reverts. You paid gas for nothing.</p>

<p>I watched someone lose 2 ETH in gas fees chasing a $500 arbitrage that existed for exactly two blocks. They didn't set <code>amountOutMinimum</code> correctly—they assumed conditions wouldn't change. They did.</p>

<p>The fix: use an aggregator like 1Inch or Matcha. Let them route through multiple hops. Their algorithms beat yours.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Off-chain: Calculate what you actually need with current market data</span>
<span class="kd">const</span> <span class="nx">getRequiredProfit</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">amount</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">quote</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://api.1inch.io/v5.0/1/quote</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">fromTokenAddress</span><span class="p">:</span> <span class="nx">USDC</span><span class="p">,</span>
        <span class="na">toTokenAddress</span><span class="p">:</span> <span class="nx">WETH</span><span class="p">,</span>
        <span class="na">amount</span><span class="p">:</span> <span class="nx">amount</span><span class="p">,</span>
        <span class="na">slippage</span><span class="p">:</span> <span class="mi">1</span>
    <span class="p">});</span>

    <span class="k">return</span> <span class="nx">quote</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">toAmount</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>2. Reentrancy (Still Bites People)</strong></p>

<p>Flash loans call your contract code. If your contract calls external contracts, you can get reentered. I'm not exaggerating—I reviewed code where someone tried to liquidate a position, got reentered mid-liquidation, and got liquidated themselves.</p>

<p>The protection: <code>nonReentrant</code> guards, checks-effects-interactions pattern, or proper state updates before external calls.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>contract SaferFlashBorrower is IFlashLoanReceiver, ReentrancyGuard {
    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external nonReentrant override returns (bytes32) {
        // nonReentrant ensures this can't be called again until completion
        // Your logic here is protected

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }
}
</code></pre>

</div>



<p><strong>3. Oracle Manipulation (The Big One)</strong></p>

<p>Here's the attack that still keeps me up at night: bZx #1 from 2020.</p>

<p>Someone used a flash loan to borrow massive amounts of asset X, crashing its price on a small DEX. A price oracle reads from that DEX and reports a lower price for X. The attacker's collateral looks better. They borrow more stables than they should. Transaction completes, oracle resets, they've stolen funds.</p>

<p>It doesn't need huge flash loan amounts. It just needs tiny DEX liquidity. Most protocols have fixed this now, but not all.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// BAD: Reading price from a single DEX
contract VulnerableOracle {
    function getPrice(address token) external view returns (uint256) {
        uint256 price = UniswapV2Library.getAmountsOut(1 ether, path)[1];
        return price; // Flash lender can manipulate this in one block
    }
}

// BETTER: Multiple oracles, time-weighted, with guards
contract SaferOracle {
    function getPrice(address token) external view returns (uint256) {
        // Time-weighted average price resists single-block manipulation
        uint256 priceTWAP = getPriceTWAP(token);
        uint256 currentPrice = getSpotPrice(token);

        // Revert if current price deviates too much from TWAP
        require(
            currentPrice &lt; priceTWAP * 110 / 100,
            "Price oracle manipulation detected"
        );

        return priceTWAP;
    }
}
</code></pre>

</div>



<p>Use Chainlink oracles. Use Balancer's rate providers. Build time-weighted comparisons. Don't be the next headline.</p>

