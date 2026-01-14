---
Title: QPI v2: Evolving QR Codes into Trustworthy Physical Data Containers
Description: 
Author: Colten Jacob
Date: 2026-01-14T20:53:58.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Colten Jacob (2026)</em></p>

<p><strong>Abstract: Building on the Foundation</strong></p>

<p>Let's be honest—QR codes are underutilized. We've built this remarkable global infrastructure for machine-readable symbols, with error correction baked right into the standard, and we use them almost exclusively as glorified hyperlinks. QPI v1 asked the provocative question: "What if the QR code wasn't a link, but the data itself?" This paper takes that question and runs with it, addressing every practical limitation head-on.</p>

<p>The original QPI specification showed how to encode images directly in QR codes, creating self-contained digital artifacts you could hold in your hand. But the community had questions: "What about larger images?" "How do we prevent tampering?" "Can this actually work in the real world?"</p>

<p>QPI v2 answers: "Yes, and here's exactly how." This isn't just theoretical—it's a complete, robust specification you could implement this week. We'll explore multi-QR chunking, integrated verification, compression optimized for the QR environment, and even tie into the fascinating "Trust on First Scan" model to create something genuinely new: trustworthy physical data.</p>

<p><strong>1. Introduction: From Clever Hack to Robust Standard</strong></p>

<p>The first QPI paper established something important: QR codes can be containers. This simple shift in perspective opens doors to applications we haven't even imagined yet. A printed menu that's machine-verifiable. A concert ticket that contains its own artwork and verification. An ID card where the photo is stored right there in the barcode.</p>

<p>But as with any good idea, the devil's in the details. A single QR code has limited capacity (~3KB max). Real-world images need more space. Real-world applications need security. Real-world users need reliability.</p>

<p>QPI v2 addresses these challenges with elegant solutions that respect the constraints of the medium. We're not trying to turn QR codes into something they're not—we're discovering what they could be with careful engineering.</p>

<p><strong>2. The Core Insight: Physical Data has Different Rules</strong></p>

<p>Before we dive into the technical details, let's establish a philosophical principle: Physical data transmission has different constraints than network transmission.</p>

<p>On the internet, we assume:</p>

<p>· Essentially unlimited bandwidth<br>
· Bidirectional communication<br>
· Reliable delivery<br>
· Server-side processing</p>

<p>In the physical world with QR codes:</p>

<p>· Each scan is a one-shot transmission<br>
· No retries unless you rescan<br>
· Error correction is baked in but finite<br>
· The medium itself can be damaged</p>

<p>QPI v2 is designed for this reality from the ground up.</p>

<p><strong>3. Chunking: Breaking the 3KB Barrier</strong></p>

<p><em>3.1 The Problem Everyone Noticed</em></p>

<p>Let's address the elephant in the room first. The original QPI paper showed that a single QR code could store small images: maybe a 32×32 RGB logo or an 80×80 grayscale portrait. Useful, but limited.</p>

<p>But what if we don't limit ourselves to one QR code?</p>

<p><em>3.2 Distributed QPI: A Simple Yet Powerful Idea</em></p>

<p>Imagine a large image split across multiple QR codes, like a puzzle. Each piece contains:</p>

<ol>
<li>A chunk of the image data</li>
<li>Metadata about its position</li>
<li>Information about the whole image</li>
</ol>

<p>Suddenly, our capacity is N × 3KB, where N is the number of QR codes. A poster with 16 QR codes could store ~48KB of data—enough for a 256×256 color image with compression.</p>

<p><em>3.3 The Chunking Protocol (Simple Version)</em></p>

<p>Here's how it works in practice:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Image Identifier: 4 bytes]
[Chunk Number: 2 bytes] 
[Total Chunks: 2 bytes]
[Image Data: Up to ~2900 bytes]
</code></pre>

</div>



<p>Why this works beautifully:</p>

<ol>
<li>Order doesn't matter—scan chunks in any sequence</li>
<li>Partial recovery is possible—get most of the image even if some chunks are missing</li>
<li>Progressive rendering—display what you have as you scan</li>
</ol>

<p><em>3.4 Real-World Example: The Restaurant Menu</em></p>

<p>Consider a restaurant menu encoded as QPI:</p>

<p>· QR Code 1: Header, restaurant info, first category (appetizers)<br>
· QR Code 2: Second category (entrees)<br>
· QR Code 3: Third category (desserts), contact info<br>
· QR Code 4: High-resolution logo, digital signature</p>

<p>Printed on a table tent, customers can scan any QR to get partial info, or scan all four to get the complete digital menu. The restaurant can update by reprinting—no app updates, no server maintenance.</p>

<p><strong>4. Trust and Verification: Beyond Simple Storage</strong></p>

<p><em>4.1 The Trust Problem</em></p>

<p>If anyone can create a QPI image, how do we know it's legitimate? A counterfeiter could create a fake ticket QPI just as easily as a real one.</p>

<p><em>4.2 Introducing "Trust on First Scan" Integration</em></p>

<p>In a separate article, I explored the "Trust on First Scan" model—a system where the first scan of a physical code establishes a baseline of trust. QPI v2 can integrate this elegantly:</p>

<p>How it works:</p>

<ol>
<li>Initial trusted scan connects to a verification server (online)</li>
<li>Server returns a cryptographic proof tied to that specific QPI</li>
<li>This proof is cached locally</li>
<li>Future scans can be verified offline against this cached proof</li>
</ol>

<p>For QPI, this means:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Standard QPI Image Data]
[Optional: Hash of image data]
[Optional: Cryptographic signature]
[Optional: "Trust Anchor" identifier]
</code></pre>

</div>



<p>The trust anchor could be:</p>

<p>· A public key distributed with the app<br>
· A hash registered on a blockchain<br>
· A certificate from a known authority</p>

<p><em>4.3 Practical Security Levels</em></p>

<p>Not every QPI needs NSA-level security. We define tiers:</p>

<p>Level 1: Integrity Only</p>

<p>· Contains SHA-256 hash of the image<br>
· Ensures the image hasn't been altered since creation<br>
· No authentication of source</p>

<p>Level 2: Source Authentication</p>

<p>· Contains digital signature<br>
· Requires public key distribution<br>
· Proves who created it</p>

<p>Level 3: Time-Bound Trust</p>

<p>· Includes timestamp and expiration<br>
· Prevents replay of old versions<br>
· Useful for tickets, certificates</p>

<p>Level 4: First-Scan Trust</p>

<p>· Integrates with Trust on First Scan model<br>
· Initial online verification caches trust<br>
· All future verifications work offline</p>

<p><em>4.4 The Beautiful Part: Backward Compatibility</em></p>

<p>The security data sits after the image data in the byte stream. A simple QPI v1 decoder will:</p>

<ol>
<li>Read the image successfully</li>
<li>Ignore the extra bytes it doesn't understand</li>
<li>Display the image (without verification)</li>
</ol>

<p>A QPI v2 decoder with security support will:</p>

<ol>
<li>Read the image</li>
<li>Detect security footer</li>
<li>Perform verification</li>
<li>Show verification status to user</li>
</ol>

<p>This means security can be added incrementally without breaking existing deployments.</p>

<p><strong>5. Compression: Making Every Byte Count</strong></p>

<p><em>5.1 The Reality of 3KB</em></p>

<p>When you only have 3,000 bytes to work with, every byte matters. A naive RGB image encoding wastes space. QPI v2 includes intelligent compression strategies:</p>

<p><em>5.2 Mode-Specific Optimization</em></p>

<p>For photographs (natural images):</p>

<p>· Convert to YCbCr color space<br>
· Chroma subsampling (4:2:0)<br>
· DEFLATE compression<br>
· Result: ~60% smaller than naive RGB</p>

<p>For logos and graphics:</p>

<p>· Optimal palette selection (median cut algorithm)<br>
· Run-length encoding for large flat areas<br>
· Result: Often 80-90% smaller than RGB</p>

<p>For text and diagrams:</p>

<p>· Convert to 1-bit black and white<br>
· Use specialized compression (CCITT Group 4)<br>
· Result: A full letter-sized page in ~5KB</p>

<p><em>5.3 Progressive Encoding: Graceful Degradation</em></p>

<p>Here's a clever trick: encode multiple quality levels in the same data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Header: 16 bytes]
[Thumbnail: 200 bytes - 32×32 grayscale]
[Medium quality: 800 bytes - adds detail]
[Full quality: remaining bytes]
</code></pre>

</div>



<p>If you only scan one QR, you get the thumbnail. Scan more, quality improves. This is perfect for:</p>

<p>· Museum exhibits (quick scan for basic info, full scan for details)<br>
· Product labels (basic info visible immediately, specs with full scan)<br>
· Emergency information (critical info first, details if available)</p>

<p><strong>6. Error Resilience: Expecting Real-World Conditions</strong></p>

<p><em>6.1 QR Codes Are Surprisingly Robust</em></p>

<p>The Reed-Solomon error correction in QR codes is excellent—up to 30% of the code can be damaged and still read correctly. But QPI v2 adds additional layers:</p>

<p><em>6.2 Cross-Chunk Parity</em></p>

<p>Similar to RAID 5 for hard drives or PAR files for Usenet, we can add parity chunks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Image Data: [Chunk 1] [Chunk 2] [Chunk 3]
Parity Data: [Parity 1] = Chunk 1 ⊕ Chunk 2
             [Parity 2] = Chunk 2 ⊕ Chunk 3
</code></pre>

</div>



<p>If any one chunk is missing, it can be reconstructed from the others. This means:</p>

<p>· Print the image chunks on the front of a document<br>
· Print parity chunks on the back<br>
· Even if part is damaged, full recovery is possible</p>

<p><em>6.3 Checksums at Multiple Levels</em></p>

<ol>
<li>Byte-level: QR's built-in error correction</li>
<li>Chunk-level: CRC32 for each chunk</li>
<li>Image-level: SHA-256 for complete image</li>
<li>Optional: Error-correcting codes within the image data itself</li>
</ol>

<p><strong>7. The QPI v2 File Format: Complete Specification</strong></p>

<p><em>7.1 Header (16 bytes - fixed)</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Bytes 0-3:   Magic number "QPI2" (0x51 0x50 0x49 0x32)
Byte 4:      Version (0x02)
Byte 5:      Flags (bit field)
              Bit 0: Compression enabled
              Bit 1: Security footer present
              Bit 2: Multi-chunk mode
              Bit 3: Progressive encoding
              Bit 4: Palette optimized
              Bits 5-7: Reserved
Bytes 6-7:   Image width (big-endian)
Bytes 8-9:   Image height (big-endian)
Bytes 10-11: Total chunks (1 = single QR)
Bytes 12-13: This chunk index (0-based)
Bytes 14-15: Payload length (bytes in this chunk)
</code></pre>

</div>



<p><em>7.2 Color Mode Byte</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>0x00: Reserved
0x01: Grayscale (8-bit)
0x02: RGB (24-bit)
0x03: Indexed-16 (4-bit palette)
0x04: Indexed-256 (8-bit palette)
0x05: Black/White (1-bit)
0x06: YCbCr 4:2:0 (12-bit effective)
0x07-0xFF: Reserved
</code></pre>

</div>



<p><em>7.3 Palette Section (if indexed modes)</em></p>

<p>Immediately follows color mode byte:</p>

<p>· Each palette entry: 3 bytes (R, G, B)<br>
· Indexed-16: 48 bytes total<br>
· Indexed-256: 768 bytes total</p>

<p><em>7.4 Compression Header (if compression enabled)</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Byte 0: Compression type
          0x01: DEFLATE (zlib compatible)
          0x02: QPI-RLE (simple run-length)
          0x03: CCITT G4 (for 1-bit images)
Bytes 1-2: Uncompressed size (big-endian)
</code></pre>

</div>



<p><em>7.5 Image Data</em></p>

<p>· Stored row by row, left to right<br>
· Format depends on color mode<br>
· May be compressed</p>

<p><em>7.6 Security Footer (if enabled)</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Byte 0: Security type
          0x01: SHA-256 only
          0x02: ECDSA signature (64 bytes)
          0x03: Timestamp + signature (72 bytes)
          0x04: Trust anchor reference
Bytes 1-2: Security data length
Bytes 3+:  Security data
</code></pre>

</div>



<p><strong>8. Implementation: Making It Real</strong></p>

<p><em>8.1 Decoder Flow</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Camera → QR Detection → Byte Extraction → 
Is QPI? → Yes → Parse Header → 
Multi-chunk? → Wait for/assemble chunks → 
Decompress → Verify Security → 
Render Image + Verification Status
</code></pre>

</div>



<p><em>8.2 Performance Considerations</em></p>

<p>On modern smartphones:</p>

<p>· Single QPI decode: &lt; 100ms<br>
· Multi-chunk assembly (4 chunks): ~300ms<br>
· Security verification: &lt; 50ms<br>
· Total user experience: Feels instantaneous</p>

<p>Memory usage:</p>

<p>· Decoding a 48KB image (16 chunks): &lt; 2MB RAM<br>
· Suitable for even low-end devices</p>

<p><strong>9. Killer Applications: Why This Matters</strong></p>

<p><em>9.1 Self-Verifying Physical Documents</em></p>

<p>Imagine a university diploma that contains:</p>

<p>· The official text<br>
· The graduate's photo<br>
· The registrar's digital signature<br>
· A link to the online verification (optional)</p>

<p>All in a single scannable area. No separate barcode, no QR that points somewhere else—the credential is the data.</p>

<p><em>9.2 Tamper-Evident Packaging</em></p>

<p>Pharmaceutical companies could print QPI codes on medication packaging containing:</p>

<p>· Legitimate product photo<br>
· Batch number and expiration<br>
· Manufacturer's signature</p>

<p>Counterfeiters would need to replicate both the visible packaging and the digital signature—much harder than just copying the box design.</p>

<p><em>9.3 Disaster-Resilient Information</em></p>

<p>In areas with poor connectivity:</p>

<p>· Emergency procedures<br>
· Maps of evacuation routes<br>
· Contact information for aid organizations</p>

<p>Printed on walls, distributed on cards. Always available, always verifiable, never dependent on servers.</p>

<p><em>9.4 Digital-Physical Artifacts</em></p>

<p>An artist creates a digital piece, then prints it as QPI. The print contains:</p>

<p>· The artwork itself (as thumbnail or full-res if multi-QR)<br>
· Artist's statement<br>
· Certificate of authenticity<br>
· Link to NFT or digital original</p>

<p>The physical print becomes a first-class digital object, not just a reproduction.</p>

<p><strong>10. Challenges and Limitations</strong></p>

<p>Let's be transparent about the trade-offs:</p>

<ol>
<li><p>Capacity is still finite<br>
Even with chunking,we're talking kilobytes, not megabytes. QPI won't replace JPEG for family photos. It's for purpose-sized images.</p></li>
<li><p>Scanner compatibility<br>
While any QR scanner can read the bytes,only QPI-aware software will interpret them as images. This requires app installation.</p></li>
<li><p>Print quality matters<br>
Low-resolution printers,poor contrast, or reflective surfaces can reduce scanning reliability.</p></li>
<li><p>Security is only as good as key management<br>
If private keys are compromised,the verification breaks. This isn't unique to QPI—it's true of all digital signatures.</p></li>
</ol>

<p><strong>11. The Future: Where Could This Go?</strong></p>

<p>QPI v3 might explore:</p>

<p>· Color QR integration: Using colored modules for higher density (once color QR standards mature)<br>
· Animated QPI: Scanning multiple codes in sequence creates animation<br>
· Interactive QPI: Codes that change based on context or user interaction<br>
· 3D QPI: Using paper texture or special inks for additional data layers</p>

<p>But here's the exciting part: QPI v2 is implementable today with existing technology. No new hardware, no standards committee approval (though that would be nice), just clever use of what we already have.</p>

<p><strong>12. Conclusion: A New Relationship with Physical Data</strong></p>

<p>QPI began with a simple but radical idea: QR codes can be destinations, not just directions. QPI v2 builds on that foundation to create something genuinely useful: a way to store and verify data in the physical world with digital precision.</p>

<p>The implications are profound. We're talking about:</p>

<p>· Democratizing data storage—anyone with a printer can create permanent, verifiable data<br>
· Bridging digital and physical—creating objects that live in both worlds seamlessly<br>
· Enabling offline trust—verification without connectivity</p>

<p>Most importantly, QPI respects the constraints of its medium. It doesn't try to turn QR codes into something they're not. Instead, it discovers what they could be with careful design and a bit of creativity.</p>

<p>This is more than a technical specification—it's an invitation. An invitation to reimagine how data can live in the physical world. To create artifacts that are both human-readable and machine-verifiable. To build systems that work even when the network doesn't.</p>

<p>The code is waiting to be written. The applications are waiting to be built. The future of physical data starts with a simple scan.</p>




<p>For queries and recommendations : <a href="mailto:coltenjacob2008@gmail.com">coltenjacob2008@gmail.com</a></p>

<p>Related Article (QPI v1) : <a href="https://dev.to/colten_jacob_10ec3150930b/qpi-qr-pixel-imaging-4el7">https://dev.to/colten_jacob_10ec3150930b/qpi-qr-pixel-imaging-4el7</a></p>

<p>Related Article (TOFS Model) : <a href="https://dev.to/colten_jacob_10ec3150930b/making-qr-codes-safe-by-design-a-trust-on-first-scan-model-tofsa-8-46i4">https://dev.to/colten_jacob_10ec3150930b/making-qr-codes-safe-by-design-a-trust-on-first-scan-model-tofsa-8-46i4</a></p>

