---
Title: FPGA_Part_03
Description: 
Author: Ahmet Kargı
Date: 2025-11-05T22:03:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>En son postta yüklediğim göselde ne olduğunu ve nasıl çalıştığını bilmediğim MUX lar hakkında araştırma yaptım ve seçenek seçici gibi düşünebiliriz. 2x1, 4x1, 8x1... gibi artan türleri var. Aslında binary bit sistemine göre seçeneği seçiyor. Örnek vermek gerekirse 4x1 MUX un 2 tane seçim pini var ve bunlarla 00 01 10 11 kombinosyanları ile I0, I1, I2, I3 inputlarından istediğimizi çıkışa verebiliyoruz.<br>
 Mantıksal işlemler yapmamızda da tabii ki işe yarıyorlar. Belli mintermleri sağlayan fonksiyonları uygulamamızı sağlıyorlar. DEMUX ise MUX un tam tersi oluyor.<br>
 Aslında projenin amacı FPGA üzerine çalışmaktı biliyorum ama digital desing üzerine olan cahilliğimi bitirmeden bu işe girmenin fayda sağlamayacağını düşünmeye başladım. O yüzden MUX öğrenmek için başladığım NESO Academy nin Digital Electronics ders playlistini izlemeye başladım. Oradan öğrendiklerimi gene FPGA başlığı altında postlicam şimdilik.<br>
Bazı şemalar:<br>
2x1 MUX:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F782cqs0n48jm68t0yvf2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F782cqs0n48jm68t0yvf2.png" alt=" " width="800" height="214"></a></p>

<p>4x1 MUX:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ax4ftr141fhz7xbv9o3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ax4ftr141fhz7xbv9o3.png" alt=" " width="600" height="423"></a></p>

<p>8x1 MUX:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frxcqvvaprd6plg70k3bd.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frxcqvvaprd6plg70k3bd.jpg" alt=" " width="800" height="594"></a></p>

<p>NESO Academy'nin içeriğinden MUX ları fonksiyonlarda kullanmayı anlatan ders:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc4zoe9rm2u2aw8p5ro66.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc4zoe9rm2u2aw8p5ro66.png" alt=" " width="800" height="214"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F41yz7oxdz2cwjdob2e83.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F41yz7oxdz2cwjdob2e83.png" alt=" " width="800" height="470"></a></p>

<p><a href="https://youtube.com/playlist?list=PLBlnK6fEyqRjMH3mWf6kwqiTbT798eAOm&amp;si=m0V2P9RXR89oHvWL" rel="noopener noreferrer">NESO Academy'nin Playlist linki</a></p>

