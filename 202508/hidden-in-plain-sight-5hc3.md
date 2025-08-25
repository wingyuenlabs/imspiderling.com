---
Title: Hidden in Plain Sight - Steganography
Description: 
Author: Michael Z
Date: 2025-08-25T20:07:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>To us, a flower looks like a splash of color. To a bee, it’s a visual map. Many flowers have ultraviolet patterns invisible to humans but clear to pollinators. These markings act like landing guides, directing bees straight to nectar.</p>

<p>UV flowers, insects disguised as leaves, octopuses blending into coral... Secrets can sit in the open, plain and boring, until you know the key that reveals their hidden meaning.</p>

<p>Nature has been “hiding” information this way for millions of years. Humans, too, learned the same trick: the best way to conceal a message is not just behind locks or codes, but inside something that looks completely ordinary.</p>

<p>The Romans, for example, wrote with milk that dried invisible, leaving a letter that looked blank until heated. To most, it was an innocent sheet of parchment, but to the intended recipient, it revealed a secret.</p>

<p>Over the centuries, methods like this grew more sophisticated, and with the rise of the digital age, steganography found entirely new opportunities.</p>

<p>Today, billions of images are viewed online every day and some of them may be carrying secrets. Let’s explore the fascinating world of steganography.</p>




<p>Steganography comes in many forms and, similarly to encryption, relies on the sender and receiver agreeing on a method. For example, "the first letter of each word reveals the password."</p>

<p>The difference with cryptography is this: with encryption, you know there’s a hidden message. With steganography, you may be completely unaware that one exists.</p>

<p>For instance, hiding the password "SynwiL" using cryptography might produce something like <code>4MJJ0NwTSiJJdBp+g9Skrw==</code> whereas steganography could encode it in a sentence like: <code>See you next week in London.</code></p>

<p>But humans are clever, and simple text-based methods can be easily uncovered.<br>
We need something bigger, way bigger, where tiny bits can be changed without anyone noticing. Painters sometimes hide their signature in the eyes of a portrait, perhaps we can do something similar with digital images.</p>

<p>Large images come with many pixels, and pixels come with many bytes.</p>

<p>For example, a 1920x1080 image has over 2 million pixels. A 24-bit color BMP image uses 3 bytes (red, green, blue) per pixel, totaling over 6 million bytes.</p>

<p>Each pixel can display 16.7 million different colors (2^(3*8)) and the human eye cannot distinguish between subtle differences. This is exactly what we can exploit: by changing the least significant bit (LSB) of a byte, we can embed a secret message in an image.</p>

<p>However, there are a few precautions:</p>

<ul>
<li>Tools can detect statistical anomalies, so it’s wise to first encrypt the message. This ensures that even if discovered, the message remains unreadable.</li>
<li>More sophisticated algorithms exist to mitigate these risks, but we won’t cover them today.</li>
</ul>

<p>Choosing which bits to flip is also crucial. Sequential changes are too obvious, and the receiver still needs to reconstruct the message. This rules out simple methods and true randomness, leaving us with pseudo-randomness. Using a PRNG (pseudo-random number generator) with a shared seed and tool, both sender and receiver can deterministically select the same seemingly random bytes. This is powerful: even if the seed is compromised, the algorithm to generate the numbers remain a mystery.</p>

<p>But enough with theory, let's see how this looks in action.</p>
<h2>
  
  
  Embedding
</h2>

<p>To embed the message into a file, we first encrypt it, and then turn it into the individual bits.<br>
We also wrap the code in a special signature so we can identify when the message is fully read when extracting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">var</span> <span class="n">BMP_HEADER_BYTES</span> <span class="o">=</span> <span class="m">54</span> <span class="c">// we must not change these</span>
<span class="k">var</span> <span class="n">START</span> <span class="o">=</span> <span class="s">"1::"</span>
<span class="k">var</span> <span class="n">END</span> <span class="o">=</span> <span class="s">":&gt;0$"</span>

<span class="n">encryptedPayload</span> <span class="o">:=</span> <span class="n">encrypt</span><span class="p">(</span><span class="n">payload</span><span class="p">)</span>
<span class="c">// wrap payload in start and end tags</span>
<span class="n">payloadBytes</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="n">START</span> <span class="o">+</span> <span class="n">encryptedPayload</span> <span class="o">+</span> <span class="n">END</span><span class="p">)</span>

<span class="c">// turn payload into bits</span>
<span class="k">var</span> <span class="n">bitStringBuilder</span> <span class="n">strings</span><span class="o">.</span><span class="n">Builder</span>
<span class="n">bitStringBuilder</span><span class="o">.</span><span class="n">Grow</span><span class="p">(</span><span class="nb">len</span><span class="p">(</span><span class="n">payloadBytes</span><span class="p">)</span> <span class="o">*</span> <span class="m">8</span><span class="p">)</span>
<span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">b</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">payloadBytes</span> <span class="p">{</span>
  <span class="n">bitStringBuilder</span><span class="o">.</span><span class="n">WriteString</span><span class="p">(</span><span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"%08b"</span><span class="p">,</span> <span class="n">b</span><span class="p">))</span>
<span class="p">}</span>
<span class="n">payloadBits</span> <span class="o">:=</span> <span class="n">bitStringBuilder</span><span class="o">.</span><span class="n">String</span><span class="p">()</span>

<span class="c">// check image is large enough</span>
<span class="n">availableBits</span> <span class="o">:=</span> <span class="nb">len</span><span class="p">(</span><span class="n">file</span><span class="p">)</span> <span class="o">-</span> <span class="n">BMP_HEADER_BYTES</span>
<span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">payloadBits</span><span class="p">)</span> <span class="o">&gt;</span> <span class="n">availableBits</span> <span class="p">{</span>
  <span class="n">log</span><span class="o">.</span><span class="n">Fatalf</span><span class="p">(</span><span class="s">"payload is too large to embed in this image."</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>"payloadBits" is now a string that has the entire payload encrypted and encoded in binary.</p>

<p>We now initialize our "random" number generator. It returns a function that returns the next position for the given seed and file. We look more in depth at this later.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">getNextPosition</span> <span class="o">:=</span> <span class="n">startRng</span><span class="p">(</span><span class="n">seed</span><span class="p">,</span> <span class="n">file</span><span class="p">)</span>
</code></pre>

</div>



<p>Now we loop through each bit of the payload, take the byte from the file at the determined position, and replace its final bit with our new payload bit.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">bit</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">payloadBits</span> <span class="p">{</span>
  <span class="n">position</span> <span class="o">:=</span> <span class="n">getNextPosition</span><span class="p">()</span>
  <span class="c">// retrieve bits from the next position in the file</span>
  <span class="n">binaryStr</span> <span class="o">:=</span> <span class="n">byteToBinaryString</span><span class="p">(</span><span class="n">file</span><span class="p">[</span><span class="n">position</span><span class="p">])</span>
  <span class="c">// replace LSB</span>
  <span class="n">newBits</span> <span class="o">:=</span> <span class="n">binaryStr</span><span class="p">[</span><span class="o">:</span><span class="nb">len</span><span class="p">(</span><span class="n">binaryStr</span><span class="p">)</span><span class="o">-</span><span class="m">1</span><span class="p">]</span> <span class="o">+</span> <span class="kt">string</span><span class="p">(</span><span class="n">bit</span><span class="p">)</span>
  <span class="c">// write bits back to the file</span>
  <span class="n">file</span><span class="p">[</span><span class="n">position</span><span class="p">]</span> <span class="o">=</span> <span class="n">binaryStringToByte</span><span class="p">(</span><span class="n">newBits</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>We write the file again and that's all there is for for embedding.</p>

<h2>
  
  
  Extracting
</h2>

<p>To extract the message again, we do the same process in reverse.</p>

<p>In an infinite loop, we go through each position using the same rng function and collect the bits. Each 8 bits, we can save one byte. Once we reach the END marker, we break the loop.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">getNextPosition</span> <span class="o">:=</span> <span class="n">startRng</span><span class="p">(</span><span class="n">seed</span><span class="p">,</span> <span class="n">file</span><span class="p">)</span>

<span class="n">byteBuffer</span> <span class="o">:=</span> <span class="s">""</span>
<span class="k">var</span> <span class="n">payloadBytes</span> <span class="p">[]</span><span class="kt">byte</span>
<span class="n">extractedBits</span> <span class="o">:=</span> <span class="m">0</span>
<span class="n">maxBitsToExtract</span> <span class="o">:=</span> <span class="nb">len</span><span class="p">(</span><span class="n">file</span><span class="p">)</span> <span class="o">*</span> <span class="m">8</span> <span class="c">// Absolute theoretical maximum</span>

<span class="k">for</span> <span class="p">{</span>
  <span class="c">// Prevent infinite loop if END marker is not found</span>
  <span class="k">if</span> <span class="n">extractedBits</span> <span class="o">&gt;</span> <span class="n">maxBitsToExtract</span> <span class="p">{</span>
    <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="s">"Extraction limit reached without finding END marker. Seed likely incorrect."</span><span class="p">)</span>
  <span class="p">}</span>

  <span class="n">position</span> <span class="o">:=</span> <span class="n">getNextPosition</span><span class="p">()</span>
  <span class="c">// extract LSB from position</span>
  <span class="n">binaryStr</span> <span class="o">:=</span> <span class="n">byteToBinaryString</span><span class="p">(</span><span class="n">file</span><span class="p">[</span><span class="n">position</span><span class="p">])</span>
  <span class="n">byteBuffer</span> <span class="o">+=</span> <span class="n">binaryStr</span><span class="p">[</span><span class="nb">len</span><span class="p">(</span><span class="n">binaryStr</span><span class="p">)</span><span class="o">-</span><span class="m">1</span><span class="o">:</span><span class="p">]</span>
  <span class="n">extractedBits</span><span class="o">++</span>

  <span class="c">// once we have one full byte...</span>
  <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">byteBuffer</span><span class="p">)</span> <span class="o">==</span> <span class="m">8</span> <span class="p">{</span>
    <span class="c">// ... append byte to our final payload</span>
    <span class="n">charByte</span> <span class="o">:=</span> <span class="n">binaryStringToByte</span><span class="p">(</span><span class="n">byteBuffer</span><span class="p">)</span>
    <span class="n">payloadBytes</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">payloadBytes</span><span class="p">,</span> <span class="n">charByte</span><span class="p">)</span>
    <span class="n">byteBuffer</span> <span class="o">=</span> <span class="s">""</span>

    <span class="n">payload</span> <span class="o">:=</span> <span class="kt">string</span><span class="p">(</span><span class="n">payloadBytes</span><span class="p">)</span>
    <span class="c">// another safety check</span>
    <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">payload</span><span class="p">)</span> <span class="o">==</span> <span class="nb">len</span><span class="p">(</span><span class="n">START</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="n">payload</span> <span class="o">!=</span> <span class="n">START</span> <span class="p">{</span>
      <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="s">"Start of the message does not match. Seed/Salt likely incorrect."</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="c">// break if end marker was detected</span>
    <span class="k">if</span> <span class="n">strings</span><span class="o">.</span><span class="n">HasSuffix</span><span class="p">(</span><span class="n">payload</span><span class="p">,</span> <span class="n">END</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">break</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
<span class="n">payload</span> <span class="o">:=</span> <span class="kt">string</span><span class="p">(</span><span class="n">payloadBytes</span><span class="p">)</span>

<span class="c">// trim start and end markers and decrypt the message</span>
</code></pre>

</div>



<h2>
  
  
  PRNG
</h2>

<p>"startRng" returns "getNextPosition". This is so it can keep track of specific variables like the counter and previous position used. "getNextPosition" creates a unique seed using the previous position and counter to avoid repeating the same positions. It also uses the image's meta data to make the positions unique for each image.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">startRng</span><span class="p">(</span><span class="n">seed</span> <span class="kt">string</span><span class="p">,</span> <span class="n">file</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">)</span> <span class="k">func</span><span class="p">()</span> <span class="kt">uint64</span> <span class="p">{</span>
    <span class="n">seen</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">(</span><span class="k">map</span><span class="p">[</span><span class="kt">uint64</span><span class="p">]</span><span class="kt">bool</span><span class="p">)</span>
    <span class="n">counter</span> <span class="o">:=</span> <span class="m">0</span>
    <span class="k">var</span> <span class="n">previousPos</span> <span class="kt">uint64</span>

    <span class="k">return</span> <span class="k">func</span><span class="p">()</span> <span class="kt">uint64</span> <span class="p">{</span>
        <span class="k">for</span> <span class="p">{</span>
            <span class="n">imageSize</span> <span class="o">:=</span> <span class="nb">len</span><span class="p">(</span><span class="n">file</span><span class="p">)</span>
      <span class="c">// create a unique seed</span>
            <span class="n">uniqueSeed</span> <span class="o">:=</span> <span class="n">seed</span> <span class="o">+</span> <span class="n">strconv</span><span class="o">.</span><span class="n">FormatUint</span><span class="p">(</span><span class="n">previousPos</span><span class="p">,</span> <span class="m">10</span><span class="p">)</span> <span class="o">+</span> <span class="s">"$#"</span> <span class="o">+</span> <span class="kt">string</span><span class="p">(</span><span class="n">file</span><span class="p">[</span><span class="o">:</span><span class="n">BMP_HEADER_SIZE</span><span class="p">])</span> <span class="o">+</span> <span class="kt">string</span><span class="p">(</span><span class="n">imageSize</span><span class="p">)</span> <span class="o">+</span> <span class="s">":"</span> <span class="o">+</span> <span class="n">strconv</span><span class="o">.</span><span class="n">Itoa</span><span class="p">(</span><span class="n">counter</span><span class="p">)</span>
            <span class="n">pos</span> <span class="o">:=</span> <span class="n">prng</span><span class="p">(</span><span class="n">uniqueSeed</span><span class="p">,</span> <span class="n">imageSize</span><span class="p">)</span>
            <span class="n">previousPos</span> <span class="o">=</span> <span class="n">pos</span>
            <span class="n">counter</span><span class="o">++</span>
      <span class="c">// safety check to avoid reusing positions</span>
            <span class="k">if</span> <span class="o">!</span><span class="n">seen</span><span class="p">[</span><span class="n">pos</span><span class="p">]</span> <span class="p">{</span>
                <span class="n">seen</span><span class="p">[</span><span class="n">pos</span><span class="p">]</span> <span class="o">=</span> <span class="no">true</span>
                <span class="k">return</span> <span class="n">pos</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c">// based on the unique seed, get the position within the available image size</span>
<span class="k">func</span> <span class="n">prng</span><span class="p">(</span><span class="n">seed</span> <span class="kt">string</span><span class="p">,</span> <span class="n">imageSize</span> <span class="kt">int</span><span class="p">)</span> <span class="kt">uint64</span> <span class="p">{</span>
    <span class="n">modifyableImageSize</span> <span class="o">:=</span> <span class="n">imageSize</span> <span class="o">-</span> <span class="n">BMP_HEADER_SIZE</span>
    <span class="n">hasher</span> <span class="o">:=</span> <span class="n">sha256</span><span class="o">.</span><span class="n">New</span><span class="p">()</span>
    <span class="n">hasher</span><span class="o">.</span><span class="n">Write</span><span class="p">([]</span><span class="kt">byte</span><span class="p">(</span><span class="n">seed</span><span class="p">))</span>
    <span class="n">seedHashStr</span> <span class="o">:=</span> <span class="n">hex</span><span class="o">.</span><span class="n">EncodeToString</span><span class="p">(</span><span class="n">hasher</span><span class="o">.</span><span class="n">Sum</span><span class="p">(</span><span class="no">nil</span><span class="p">))[</span><span class="m">0</span><span class="o">:</span><span class="m">16</span><span class="p">]</span>
    <span class="n">seedHash</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">strconv</span><span class="o">.</span><span class="n">ParseUint</span><span class="p">(</span><span class="n">seedHashStr</span><span class="p">,</span> <span class="m">16</span><span class="p">,</span> <span class="m">64</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="n">seedHash</span><span class="o">%</span><span class="kt">uint64</span><span class="p">(</span><span class="n">modifyableImageSize</span><span class="p">)</span> <span class="o">+</span> <span class="kt">uint64</span><span class="p">(</span><span class="n">BMP_HEADER_SIZE</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>






<p>It's surprisingly simple! For the full code, check out <a href="https://github.com/MZanggl/stego-go/tree/main" rel="noopener noreferrer">https://github.com/MZanggl/stego-go/tree/main</a>.</p>

