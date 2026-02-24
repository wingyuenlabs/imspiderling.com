---
Title: T-Ruby: Adding Static Typing to Ruby Without Runtime Overhead
Description: 
Author: Andrey Eremin
Date: 2026-02-24T21:50:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Static typing is a formidable tool that brings immense value to codebases of all sizes. From tiny scripts to massive monoliths, the benefits are hard to ignore: you get live documentation that is always up to date, enhanced readability, and a reliable safety net that significantly boosts code reliability.</p>

<p>Different dynamic languages have taken various paths to solve the typing puzzle. Python introduced native but optional type hints, while the JavaScript ecosystem moved toward a separate language entirely with TypeScript. Within the Ruby community, we’ve seen two distinct implementations in RBS and Sorbet, which have recently begun to converge thanks to the introduction of RBS inline and Sorbet's support for it.</p>

<p>However, the current Ruby approach isn't without its friction. For many developers, typing still feels like a matter of personal preference rather than a core requirement. We never forget to write tests because they are the heartbeat of our CI/CD pipelines, but because type checking is often seen as "extra," it is far too easy for static checks to be ignored or forgotten entirely.</p>

<p>There is a groundbreaking experiment currently gaining traction called <a href="https://type-ruby.github.io/" rel="noopener noreferrer"><strong>T-Ruby</strong></a>. Unlike standard runtime type-checking systems, T-Ruby compiles typed code down to pure, undecorated Ruby with zero runtime overhead. If you are looking for a shorthand description, T-Ruby is essentially TypeScript for the Ruby language. It acts as a layer on top of the language where you write RBS-inspired code that is eventually compiled into plain, standard Ruby.</p>

<p>While some might ask why we shouldn't just use Crystal, there is a fundamental difference in philosophy. Crystal is a separate, Ruby-like language with its own ecosystem. T-Ruby, by contrast, is an extra layer specifically designed to stay close to the Ruby we already know. Much like how TypeScript eventually outputs plain JavaScript, T-Ruby ensures your final product remains pure Ruby.</p>

<p>To demonstrate this, I’ve updated a script I originally used in 2025 to <a href="https://www.eremin.eu/blog/statictyping/static-typing-the-missing-ruby-tool" rel="noopener noreferrer">showcase RBS</a>. This is a simple HTTP client that fetches the latest Ruby release tag from GitHub. With T-Ruby, we can now embed types directly into the class structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>require "httparty"
require "json"
require "time"

class RubyVersion
  API_URL = "https://api.github.com/repos/ruby/ruby/releases/latest"

  class Response
    attr_reader :code: Integer
    attr_reader :json: Hash&lt;String, untyped&gt;


    def initialize(code: Integer, json: Hash&lt;String, untyped&gt;): nil
      @code = code
      @json = json
    end

    def success?: Boolean
      (200..299).include?(@code)
    end
  end


  def fetch_response: Response
    http = HTTParty.get(API_URL, headers: { 'User-Agent' =&gt; 'static-typing-demo' })
    Response.new(http.code.to_i, JSON.parse(http.body))
  end

  def self.fetch(printer: Proc&lt;[String, String, Time], nil&gt;): nil
    resp = new.fetch_response
    raise "HTTP #{resp.code}" unless (200..299).include?(resp.code)
    data = resp.json
    published = Time.parse((data['published_at'] || Time.now.utc.iso8601).to_s)
    printer.call((data['tag_name'] || data['name']).to_s, data['html_url'], published)
  end
end
</code></pre>

</div>



<p>By adding these types, we can strictly define the structure of our Proc and ensure the data passed through the system is valid. In standard Ruby, we would normally rely on documentation or manual guard clauses to achieve this level of certainty.</p>

<p>Setting up the environment is straightforward: a quick <code>gem install t-ruby</code> followed by <code>trc --init</code> creates a configuration file. This allows you to specify your source folders for <code>*.trb</code> files, your destination for compiled Ruby, and even post-compile commands like auto-running RSpec or Minitest.</p>

<p>The integration with Rails is currently one of the more experimental aspects. Because Rails relies on a specific folder structure, the cleanest approach seems to be storing the entire app folder within your T-Ruby source directory. Files that don't contain T-Ruby syntax are simply copied over to the destination folder unmodified. This mirrors the <code>/src</code> and <code>/dist</code> workflow common in modern JavaScript frameworks like Next.js.</p>




<p>Is the future finally here? Not quite. T-Ruby is very much in a "technical preview" phase. While the website and documentation look professional and polished, the implementation can be temperamental. During my testing, I found that even official examples could fail immediately after installation.</p>

<p>However, the potential is undeniable. While working on a small Rails app with T-Ruby, I realized I no longer needed a separate collection of RBS files. Writing types alongside business logic feels natural, and knowing the code is "type-correct" at compile time provides a unique peace of mind. It’s a bold step forward for the Ruby ecosystem even if it isn't quite ready for your production environment just yet.</p>

<p>Originally posted in <a href="https://newsletters.eremin.eu/posts/t-ruby-adding-static-typing-to-ruby-without-runtime-overhead" rel="noopener noreferrer">my blog</a>.</p>

