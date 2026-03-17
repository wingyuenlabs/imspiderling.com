---
Title: Understanding the OpenClaw Near CLI Tools Skill: A Complete Guide to Installing and Using near-cli-rs
Description: 
Author: Aloysius Chan
Date: 2026-03-17T21:48:12.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Understanding the OpenClaw Near CLI Tools Skill
</h1>

<p>The OpenClaw Near CLI Tools skill is a ready‑to‑use knowledge module that<br>
explains how to install, configure, and work with the NEAR Protocol<br>
command‑line interface, specifically the <code>near-cli-rs</code> binary. When a user<br>
asks about setting up NEAR CLI, importing accounts, deploying contracts, or<br>
checking balances, the skill provides step‑by‑step instructions,<br>
troubleshooting tips, and best‑practice recommendations. By following the<br>
guidance in this skill, developers and blockchain enthusiasts can get a<br>
functional NEAR CLI environment on Linux, macOS, Windows, or any system that<br>
supports Node.js, Rust, or a simple shell script.</p>

<h2>
  
  
  What the Skill Covers
</h2>

<p>The skill is organized into clear sections that mirror the typical workflow of<br>
a NEAR developer. It begins with an overview of why the NEAR CLI is essential,<br>
then presents four distinct installation methods: a portable shell script,<br>
npm/npx for JavaScript developers, a Windows MSI installer, and a Cargo‑based<br>
Rust installation. Each method includes prerequisites, exact commands, and<br>
notes on adding the binary to the system PATH. After installation, the skill<br>
shows how to verify the binary, view configuration locations, explore<br>
available command groups, and perform first‑time actions such as viewing an<br>
account summary, importing an existing account, or creating a new one.</p>

<p>Beyond the basics, the skill contains a troubleshooting table that addresses<br>
common errors like "command not found," permission problems, missing installer<br>
files, and slow downloads. It also points to additional resources such as the<br>
official GitHub repository, NEAR documentation, Stack Overflow tags, and the<br>
NEAR Discord community. Finally, a quick‑reference cheat sheet condenses the<br>
most useful commands into a compact format for rapid recall.</p>

<h2>
  
  
  Installation Method 1: Portable Shell Script
</h2>

<p>The shell script method is recommended for most Linux and macOS users, as well<br>
as those working inside Windows Subsystem for Linux (WSL). The script<br>
downloads the latest release of <code>near-cli-rs</code> from GitHub and executes it in<br>
one line. Because the script installs the binary into <code>$HOME/.cargo/bin</code>,<br>
users must ensure that this directory is part of their PATH. The skill<br>
explains how to temporarily add the path for the current session with <code>export<br>
PATH="$HOME/.cargo/bin:$PATH"</code> and how to make the change permanent by<br>
appending the same line to <code>~/.bashrc</code> or <code>~/.zshrc</code>. If the binary is not<br>
found after installation, the skill advises using <code>which near</code> or <code>whereis<br>
near</code> to locate the absolute path before running any NEAR command.</p>

<h2>
  
  
  Installation Method 2: npm/npx
</h2>

<p>For developers who already work with Node.js, the skill shows how to use <code>npx</code><br>
to run <code>near-cli-rs</code> without a global install, or how to install it globally<br>
with <code>npm install -g near-cli-rs</code>. The npm approach works on any platform that<br>
supports Node.js, making it a convenient option for Windows users who prefer<br>
not to deal with Rust or shell scripts. After a global npm install, the skill<br>
reminds users to verify that the npm bin directory (typically <code>$(npm config<br>
get prefix)/bin</code>) is in the PATH, and provides the appropriate export command<br>
to add it if necessary.</p>

<h2>
  
  
  Installation Method 3: Windows MSI Installer
</h2>

<p>Windows users who do not have Node.js or Rust installed can take advantage of<br>
the pre‑built MSI installer available on the NEAR CLI‑RS releases page. The<br>
skill walks through downloading the latest <code>near-cli-rs-installer-x64.msi</code>,<br>
double‑clicking the file, and following the wizard. The installer<br>
automatically adds the NEAR CLI binary to the system PATH, so no manual<br>
configuration is required. After installation, users can open a new Command<br>
Prompt or PowerShell window and run <code>near --version</code> to confirm success.</p>

<h2>
  
  
  Installation Method 4: Cargo (Rust)
</h2>

<p>Rust enthusiasts can install <code>near-cli-rs</code> directly with Cargo, either from<br>
crates.io or from the Git repository. The skill lists the prerequisite of<br>
having Rust installed via <a href="https://rustup.rs/" rel="noopener noreferrer">rustup.rs</a> and notes that<br>
certain Linux distributions may need the <code>libudev-dev</code> or <code>libudev-devel</code><br>
package for USB device support. The installation command is <code>cargo install<br>
near-cli-rs</code> or, to track the latest master branch, <code>cargo install --git<br>
https://github.com/near/near-cli-rs</code>. As with the other methods, the skill<br>
emphasizes checking that <code>$HOME/.cargo/bin</code> is in the PATH and reloading the<br>
shell configuration if needed.</p>

<h2>
  
  
  Verification
</h2>

<p>Once the installation method of choice has been completed, the skill instructs<br>
users to run <code>near --version</code>. The expected output shows something like <code>near-<br>
cli-rs 0.23.6</code> (or a newer version). If the command is not found, the skill<br>
provides a checklist: confirm the installation directory, verify that the<br>
directory is in PATH, and, if necessary, export the path manually. The skill<br>
also shows how to inspect the PATH variable with <code>echo $PATH</code> and grep for<br>
<code>cargo</code> or the npm prefix to ensure the binary location is present.</p>

<h2>
  
  
  Configuration and Help
</h2>

<p>The NEAR CLI stores its configuration in a TOML file located by default at<br>
<code>~/.config/near-cli/config.toml</code>. Users can view this path by running <code>near<br>
--help</code>, which also prints a summary of all available command groups. The<br>
skill breaks down each group: <code>account</code> for account management, <code>tokens</code> for<br>
fungible and non‑fungible tokens, <code>staking</code> for validator operations,<br>
<code>contract</code> for deploying and calling smart contracts, <code>transaction</code> for raw<br>
transaction handling, and <code>config</code> for managing network endpoints. By invoking<br>
<code>near  --help</code> (e.g., <code>near account --help</code>), users can see the specific<br>
subcommands and options available for each area.</p>

<h2>
  
  
  First Steps with the CLI
</h2>

<p>After verification, the skill guides users through common introductory tasks.<br>
To view an existing account’s summary, the command is <code>near account view-<br>
account-summary  network-config mainnet now</code>. The skill explains that the<br>
<code>network-config</code> flag selects the target network (mainnet, testnet, or a<br>
custom RPC) and that <code>now</code> requests the latest block data. Importing an<br>
account from a seed phrase or key pair is done with <code>near account import-<br>
account</code>, which launches an interactive prompts flow. Creating a brand‑new<br>
account requires an existing funded account to pay for the new account’s<br>
creation; the skill walks through the <code>near account create-account</code> command<br>
and notes the need to provide a creator account ID and optionally a reference<br>
to a faucet or funder.</p>

<h2>
  
  
  Troubleshooting Common Issues
</h2>

<p>The skill includes a dedicated troubleshooting section that maps symptoms to<br>
causes and solutions. For the classic "near: command not found" error, the<br>
remedy is to locate the binary with <code>which near</code> or <code>whereis near</code> and then<br>
add its directory to PATH, either temporarily or permanently. If a "Permission<br>
denied" message appears when running the shell script installer, the skill<br>
advises making the script executable with <code>chmod +x near-cli-rs-installer.sh</code><br>
before executing it. When the installer file seems missing, the skill suggests<br>
checking the GitHub releases page manually and using the API to fetch the<br>
exact download URL for the installer script. For slow or hanging downloads,<br>
the recommendation is to add timeout flags to curl, such as <code>--connect-timeout<br>
30 --max-time 300</code>, to avoid indefinite waiting.</p>

<h2>
  
  
  Additional Resources
</h2>

<p>To deepen understanding, the skill points learners to the official NEAR CLI‑RS<br>
GitHub repository (<a href="https://github.com/near/near-&lt;br&gt;%0Acli-rs" rel="noopener noreferrer">github.com/near/near-cli-rs</a>) where they can read the source code, inspect release notes, and<br>
contribute issues. The NEAR Protocol documentation<br>
(<a href="https://docs.near.org/" rel="noopener noreferrer">docs.near.org</a>) offers comprehensive guides on<br>
accounts, smart contracts, and token standards. For community‑driven help, the<br>
NEAR Stack Overflow tag<br>
(<a href="https://stackoverflow.com/questions/tagged/nearprotocol" rel="noopener noreferrer">stackoverflow.com/questions/tagged/nearprotocol</a>)<br>
and the NEAR Discord (<a href="https://discord.gg/near" rel="noopener noreferrer">discord.gg/near</a>) are<br>
highlighted as places to ask questions, share experiences, and stay up‑to‑date<br>
with ecosystem news.</p>

<h2>
  
  
  Quick Reference Cheat Sheet
</h2>

<ul>
<li>Install via shell script: <code>curl https://github.com/near/near-cli-rs/releases/latest/download/near-cli-rs-installer.sh | sh</code>
</li>
<li>Install via npm: <code>npm install -g near-cli-rs</code>
</li>
<li>Run without install: <code>npx near-cli-rs</code>
</li>
<li>Verify: <code>near --version</code>
</li>
<li>View account: <code>near account view-account-summary  network-config mainnet now</code>
</li>
<li>Import account: <code>near account import-account</code>
</li>
<li>Create account: <code>near account create-account</code>
</li>
<li>Help: <code>near --help</code> and <code>near  --help</code>
</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>The OpenClaw Near CLI Tools skill consolidates everything a developer needs to<br>
get started with the NEAR Protocol command‑line interface. By presenting<br>
multiple installation pathways, clear verification steps, detailed command<br>
groups, and practical first‑time actions, the skill reduces the friction<br>
associated with setting up a blockchain development environment. Whether you<br>
prefer a simple shell script, the familiarity of npm, the convenience of a<br>
Windows installer, or the control of Cargo, the skill provides the exact<br>
commands and explanations required. With the troubleshooting tips and<br>
quick‑reference cheat sheet, users can quickly resolve issues and continue<br>
building on NEAR with confidence.</p>

<p>Skill can be found at:<br>

tools/SKILL.md&gt;</p>

