---
Title: Ethereum-Solidity Quiz Q20: What is "calldata"?
Description: 
Author: MihaiHng
Date: 2026-01-14T20:37:35.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Calldata</strong> in Solidity is the read-only data passed to a smart contract function when it's called externally. It contains the encoded function selector(bytes4) and all the function arguments( encoded in bytes32).</p>

<p><strong>calldata contains:</strong></p>

<ul>
<li>Function selector (first 4 bytes) — The function's keccak256 hash truncated to 4 bytes</li>
<li>Function arguments (remaining bytes) — Encoded parameters passed to the function</li>
</ul>

<p><strong>Important:</strong></p>

<p>Use calldata for function parameters when you:</p>

<ul>
<li>Only need to read the data</li>
<li>Want to save gas by avoiding memory copies</li>
<li>Are working with arrays or strings that won't be modified</li>
</ul>

<p><strong>Calldata vs Memory vs Storage:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Location</th>
<th>Type</th>
<th>Gas Cost</th>
<th>Persistence</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td>Calldata</td>
<td>Read-only</td>
<td>Cheapest</td>
<td>Temporary</td>
<td>Function inputs</td>
<td></td>
</tr>
<tr>
<td>Memory</td>
<td>Read/Write</td>
<td>Medium</td>
<td>Temporary</td>
<td>Local variables, arrays</td>
<td></td>
</tr>
<tr>
<td>Storage</td>
<td>Read/Write</td>
<td>Most expensive</td>
<td>Permanent</td>
<td>State variables</td>
<td></td>
</tr>
</tbody>
</table></div>

