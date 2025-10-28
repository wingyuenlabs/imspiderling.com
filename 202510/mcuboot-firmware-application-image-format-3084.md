---
Title: MCUboot: Firmware Application Image Format
Description: 
Author: Ripan Deuri
Date: 2025-10-28T22:03:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>When MCUboot bootloader prepares an application for loading, it expects the image to follow a specific structure. The complete image consists of three major components arranged sequentially in memory:</p>

<ul>
<li>image header</li>
<li>application code and data, and</li>
<li>the TLV (Type-Length-Value) area that contains metadata and cryptographic signatures.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>+---------------------------+
|      Image Header         |
+---------------------------+
|    Application Code       |
|      and Data             |
|     (variable size)       |
+---------------------------+
|      TLV Area             |
|   (Protected + Info)      |
|    (variable size)        |
+---------------------------+
</code></pre>

</div>



<p>The image header sits at the very beginning of the image and contains essential metadata that MCUboot needs to validate and load the application.</p>

<p>Following the header comes the actual application (firmware) binary. This section has variable size depending on the application's complexity and requirements.</p>

<p>At the end of the image lies the TLV area, which contains structured metadata organised as type-length-value triplets. This area holds critical information including cryptographic signatures, hash values, dependency information, and other metadata needed for secure boot operations.</p>

<h2>
  
  
  Image Header Format
</h2>

<p>The image header has below fields -<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Offset    Size    Field               Description
======    ====    =====               ===========
0x00      4       ih_magic            Magic number (0x96f3b83d)
0x04      4       ih_load_addr        Load address (optional)
0x08      2       ih_hdr_size         Header size
0x0A      2       ih_protect_tlv_size Protected TLV area size
0x0C      4       ih_img_size         Image size (excl. header &amp; TLV)
0x10      4       ih_flags            Image flags
0x14      4       ih_ver              Version (major.minor.revision+build)
0x18      4       _pad1               Reserved (padding)
0x1C      4       _pad2               Reserved (padding)
</code></pre>

</div>



<p>The magic number <code>0x96f3_b83d</code> serves as the primary identifier that this is indeed a valid MCUboot image. MCUboot checks this value first before proceeding with any further validation.</p>

<p>The load address field (optional) can specify where in RAM the image should be loaded.</p>

<p>The header size field always contains 32 (0x20) for the current image format version, but this field allows for future extensibility if the header format needs to evolve.</p>

<p>Protected TLV size indicates how many bytes at the beginning of the TLV area are "protected" TLVs. These protected TLVs are included in the signature calculation.</p>

<p>The image size field contains the size of the application binary, excluding both the header and TLV areas. This allows MCUboot to know where the TLV area begins.</p>

<p>Flags control various bootloader behaviors such as whether the image is encrypted, whether it's a RAM load image, or other operational modes.</p>

<p>The version field encodes semantic versioning information in a packed 4-byte format. MCUboot uses this for anti-rollback protection and upgrade/downgrade decisions. The encoding packs major (8 bits), minor (8 bits), revision (16 bits), and a 32-bit build number into a version tuple.</p>

<h2>
  
  
  TLV Area Structure
</h2>

<p>The TLV area begins immediately after the application binary. It starts with a TLV info header that describes the TLV area itself, followed by individual TLV entries. The structure looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>+---------------------------+
|   TLV Info Header         |
|      (4 bytes)            |
|   - Magic: 0x6907         |
|   - Total Length          |
+---------------------------+
|   Protected TLVs          |
|   (if present)            |
+---------------------------+
|   TLV Info Header         |
|   for regular TLVs        |
|      (4 bytes)            |
|   - Magic: 0x6908         |
|   - Total Length          |
+---------------------------+
|   Regular TLVs            |
|   (SHA, signature, etc)   |
+---------------------------+
</code></pre>

</div>



<p>Each individual TLV entry follows this format:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Offset    Size    Field         Description
======    ====    =====         ===========
0x00      1       type          TLV type identifier
0x01      1       _pad          Padding byte (reserved)
0x02      2       len           Length of value
0x04      len     value         TLV value data
</code></pre>

</div>



<p>The TLV area can be divided into two distinct sections: <strong>protected TLVs</strong> and <strong>regular TLVs</strong>.</p>

<h2>
  
  
  Protected TLVs
</h2>

<p>Protected TLVs are special metadata entries that become part of the signed content. When MCUboot calculates or verifies a signature, it includes the protected TLVs in the hash calculation along with the header and application code. So, any modification to protected TLVs will invalidate the signature.</p>

<p>The protected TLV section begins immediately after the application code and has its own TLV info header with magic number <code>0x6907</code>. The size of this protected section is specified in the image header's <code>ih_protect_tlv_size</code> field.</p>

<p>Common protected TLVs include <strong>dependency information</strong> and <strong>security counter values</strong>.</p>

<p><strong>Example: Image with Protected TLVs</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Image Header (0x00 - 0x1F):
+----+----+----+----+----+----+----+----+
| 3D | B8 | F3 | 96 | 00 | 00 | 01 | 00 |  Magic + Load address
+----+----+----+----+----+----+----+----+
| 20 | 00 | 10 | 00 | 00 | 40 | 00 | 00 |  Hdr size, Prot TLV size, Img size
+----+----+----+----+----+----+----+----+
| 00 | 00 | 00 | 00 | 01 | 02 | 03 | 04 |  Flags + Version
+----+----+----+----+----+----+----+----+
| 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |  Padding
+----+----+----+----+----+----+----+----+

Application Code (0x20 - 0x401F):
[... 16384 bytes of application code ...]

Protected TLV Area (0x4020 - 0x402F):
+----+----+----+----+----+----+----+----+
| 07 | 69 | 0C | 00 | 30 | 00 | 04 | 00 |  Magic 0x6907, len=12, Type=DEPENDENCY (0x30)
+----+----+----+----+----+----+----+----+
| 05 | 00 | 00 | 00 | 01 | 00 | 00 | 00 |  Dep: img_id=5, version=1.0.0.0
+----+----+----+----+----+----+----+----+

Regular TLV Area (0x4030 onwards):
+----+----+----+----+----+----+----+----+
| 08 | 69 | 44 | 00 | 10 | 00 | 20 | 00 |  Magic 0x6908, len=68, Type=SHA256 (0x10)
+----+----+----+----+----+----+----+----+
| A3 | 7F | 2E | 9B | ... 32 bytes ...  |  SHA256 hash value
</code></pre>

</div>



<p>In this example, the protected TLV area contains a dependency entry (type 0x30) that specifies this image requires image ID 5 with a minimum version of 1.0.0.0 to be present. Since this dependency information is in the protected TLV area, it's included when calculating the hash for signature verification. This prevents an attacker from removing dependency requirements.</p>

<h2>
  
  
  Image Signing Coverage
</h2>

<p>The signature calculation in MCUboot follows a specific order and includes specific sections of the image.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>+---------------------------+
|      Image Header         |  ← INCLUDED in signature
|        (32 bytes)         |
+---------------------------+
|                           |
|    Application Code       |  ← INCLUDED in signature
|      and Data             |
|                           |
+---------------------------+
|   Protected TLV Info      |  ← INCLUDED in signature
|      (4 bytes)            |
+---------------------------+
|   Protected TLV Entries   |  ← INCLUDED in signature
+---------------------------+
|   Regular TLV Info        |  ← NOT included
|      (4 bytes)            |
+---------------------------+
|   SHA256 Hash TLV         |  ← NOT included
+---------------------------+
|   Signature TLV           |  ← NOT included
+---------------------------+
|   Other TLVs              |  ← NOT included
+---------------------------+
</code></pre>

</div>



<p>The signature is calculated over a contiguous region starting from byte 0 (the image header) and extending through the protected TLV area. The exact calculation process follows these steps:</p>

<p>First, MCUboot initializes a hash context (SHA256 for most signature algorithms). Then it feeds data into the hash function in this exact order: the complete 32-byte image header, the entire application code section as specified by <code>ih_img_size</code>, and if <code>ih_protect_tlv_size</code> is non-zero, the protected TLV area including its info header and all protected TLV entries.</p>

<p>The resulting hash is then signed using the private key corresponding to your chosen signature algorithm. This signature is stored as a TLV entry in the regular TLV area, along with the SHA256 hash itself.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Offset: 0x0000 - Image Header
3D B8 F3 96 00 00 01 00 20 00 10 00 00 30 00 00
00 00 00 00 01 02 03 04 00 00 00 00 00 00 00 00

Offset: 0x0020 - Application starts
E7 FE FF FF 00 48 85 46 00 F0 12 F8 FE E7 ...
[... continues for 12288 bytes ...]

Offset: 0x3020 - Protected TLV area starts
07 69 0C 00 30 00 04 00 05 00 00 00 01 00 00 00

Offset: 0x302C - Regular TLV area starts
08 69 48 01 10 00 20 00  ← TLV Info: magic=0x6908, len=328
                            (for SHA + RSA2048 sig)

Offset: 0x3034 - SHA256 TLV
10 00 20 00  ← Type=SHA256 (0x10), len=32
B7 3A 9D 2F 8C 4E A6 1B ... (32 bytes of hash)

Offset: 0x3058 - RSA2048 Signature TLV
20 00 00 01  ← Type=RSA2048 (0x20), len=256
9F 2E 8A 7C ... (256 bytes of signature)
</code></pre>

</div>



<p>In the above example, the hash at offset 0x3034 is computed over bytes 0x0000 through 0x302B (header + app + protected TLVs). The signature at offset 0x3058 is the RSA signature of that hash using the private key.</p>

<p>If someone modifies a byte in the application code, the header, or the protected TLV area, the SHA256 hash will change completely, and signature verification will fail. However, non-protected TLVs can be added or modified without breaking the signature, as they come after the signed region.</p>

<h2>
  
  
  MCUboot Image Validation Process
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>+------------------+
|   MCUboot Start  |
+------------------+
         |
         v
+------------------+
| Read Slot 0      |
| Image Header     |
+------------------+
         |
         v
+------------------+      No
|  Magic Valid?    |---------&gt; [Try Slot 1]
+------------------+
         | Yes
         v
+------------------+
|  Parse Header    |
| Extract img_size |
| protect_tlv_size |
+------------------+
         |
         v
+------------------+
|   Locate TLVs    |
| (header + size)  |
+------------------+
         |
         v
+------------------+
| Parse TLV Info   |
| Validate Magic   |
+------------------+
         |
         v
+------------------+
| Find SHA256 TLV  |
+------------------+
         |
         v
+------------------+
| Calculate Hash   |
| Over Signed Area |
+------------------+
         |
         v
+------------------+      No
| Hash Matches?    |---------&gt; [FAIL: Invalid Image]
+------------------+
         | Yes
         v
+------------------+
| Find Signature   |
| TLV (RSA/ECDSA)  |
+------------------+
         |
         v
+------------------+
| Verify Signature |
| Using Public Key |
+------------------+
         |
         v
+------------------+      No
| Sig Valid?       |---------&gt; [FAIL: Invalid Signature]
+------------------+
         | Yes
         v
+------------------+
| Check Version    |
| Security Counter |
+------------------+
         |
         v
+------------------+
| Validate         |
| Dependencies     |
+------------------+
         |
         v
+------------------+
| Image Valid!     |
| Boot Application |
+------------------+
</code></pre>

</div>



