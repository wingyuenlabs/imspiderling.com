---
Title: awesome-trending-repos: GitHub Trending'i Otomatik Takip Etme
Description: 
Author: Furkan Köykıran
Date: 2026-03-17T21:37:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>GitHub'ta her gün yüzlerce yeni proje trending listesine giriyor. Ancak bu projeleri takip etmek, hangilerinin gerçekten önemli olduğunu belirlemek zaman alıcı bir iş. Bu sorunu çözmek için <a href="https://github.com/furkankoykiran/awesome-trending-repos" rel="noopener noreferrer">awesome-trending-repos</a> projesini geliştirdim.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" alt="GitHub Actions" width="800" height="367"></a><br>
<em>GitHub Actions ile otomatik günlük güncellemeler</em></p>

<h2>
  
  
  Proje Nedir?
</h2>

<p>awesome-trending-repos, GitHub'ın trending sayfasındaki projeleri otomatik olarak takip eden, günlük olarak güncellenen bir liste. Proje her gün gece yarısı UTC'de çalışıyor ve trending reposu analiz edip sonuçları README.md dosyasına yazıyor.</p>

<p><strong>Temel özellikler:</strong></p>

<ul>
<li>🔄 Günlük otomatik güncellemeler</li>
<li>📊 Geçmiş karşılaştırmaları ve sıralama değişiklikleri</li>
<li>📈 ASCII grafikleri ve trend görselleştirmeleri</li>
<li>🏆 En hızlı büyüyen repolar (rising stars)</li>
<li>💾 7 günlük geçmiş veri saklama</li>
</ul>

<h2>
  
  
  Nasıl Çalışır?
</h2>

<p>Proje, GitHub Actions kullanarak otomatik olarak çalışıyor. Her gün gece yarısı (UTC) bir workflow tetikleniyor, GitHub trending sayfasını scrape ediyor, verileri analiz ediyor ve sonuçları README.md dosyasına yazıyor.</p>

<p><strong>Veri toplama süreci:</strong></p>

<ol>
<li>GitHub trending sayfasını scrape etme (Cheerio ile)</li>
<li>GitHub Search API'den ek veri alma (fallback olarak)</li>
<li>Projeleri dil kategorilerine göre ayırma</li>
<li>Geçmiş verilerle karşılaştırma yapma</li>
<li>README.md'yi otomatik güncelleme</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" alt="GitHub Actions" width="800" height="367"></a><br>
<em>CI/CD pipeline ile otomatik workflow yönetimi</em></p>

<h2>
  
  
  Özellikler
</h2>

<h3>
  
  
  Günlük Otomasyon
</h3>

<p>GitHub Actions ile her gün otomatik olarak çalışıyor. Manuel hiçbir işlem gerektirmiyor. Sistem her gün gece yarısı UTC'de trending verilerini çekip README.md'yi güncelliyor.</p>

<h3>
  
  
  Geçmiş Takibi
</h3>

<p>Proje, son 7 günün verilerini saklıyor. Bu sayede bir projenin trending listesindeki konumu nasıl değişmiş görebiliyorsunuz. Yeni giren projeler, yükselenler ve düşenler hakkında detaylı bilgi veriliyor.</p>

<h3>
  
  
  Dil Bazlı Sıralama
</h3>

<p>Trending projeleri programlama diline göre kategorize ediyor. JavaScript, Python, TypeScript, Go, Rust gibi popüler dillerdeki projeleri ayrı ayrı takip edebiliyorsunuz.</p>

<h3>
  
  
  Görselleştirme
</h3>

<p>ASCII grafikleri ile projelerin yükselişini ve düşüşünü görsel olarak takip edebiliyorsunuz. Hangi projelerin kaç yıldız aldığını, hangi hızla büyüdüklerini grafiklerden görebiliyorsunuz.</p>

<h2>
  
  
  Teknik Detaylar
</h2>

<p>Proje Node.js ile yazıldı ve ES Modules kullanıyor. GitHub Actions ile otomatikleştirildi ve Octokit library'i ile GitHub API'ye entegre edildi.</p>

<p><strong>Kullanılan teknolojiler:</strong></p>

<ul>
<li>Node.js 20+</li>
<li>GitHub Actions (cron tabanlı scheduling)</li>
<li>Octokit (GitHub API client)</li>
<li>Cheerio (web scraping)</li>
<li>Axios (HTTP requests)</li>
</ul>

<h2>
  
  
  Sonuç
</h2>

<p>awesome-trending-repos projesi ile GitHub'taki trending projeleri otomatik olarak takip edebiliyorsunuz. Her gün güncellenen liste ile en yeni ve en popüler projeleri kaçırmıyorsunuz.</p>

<p>Proje hem kişisel kullanım için hem de developer community'si için değerli bir kaynak oluşturuyor. Otomasyon sayesinde zaman kazandırıyor ve trendleri takip etmeyi kolaylaştırıyor.</p>




<p><em>Bu yazı <a href="https://github.com/furkankoykiran/DevTo-MCP" rel="noopener noreferrer">DevTo-MCP</a> aracılığıyla DEV.to'ya gönderilmiştir.</em></p>

