---
Title: Automatic cross-platform testing: part 7: 32 bit, again
Description: 
Author: David Cantrell
Date: 2026-02-27T21:23:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>I have <a href="https://dev.to/drhyde/automatic-cross-platform-testing-part-6-32-bit-linux-fh2">written about this before</a>, but I've had to do it again. I noted previously that Github's actions for things like checking out a repository and for retrieving build artifacts are now 64-bit only. You've been able to work around that by using older versions of the actions but I didn't want to do that so I'm taking a different approach.</p>

<p>My new method builds on work I've done previously to pull all my testing for different Unixy platforms together into one workflow. <a href="https://github.com/DrHyde/perl-modules-Scalar-Type/blob/38e64fe918da7feee0b0e7dba39a6be00bdc0ddb/.github/workflows/install-various-OSes.yml" rel="noopener noreferrer">This</a> tests on 64-bit versions of NetBSD, FreeBSD, OpenBSD, IllumOS (that is, the modern version of Open Solaris), and Linux. And as of earlier today it also runs the tests on 32-bit Linux. As in my previous blog entry it's still really 64-bit hardware, with modern x86 ISA extensions, but it's a 32-bit OS image with 32-bit addressing, userland, compiler toolchain and so on. If you absolutely must have a pure 32-bit x86 environment then you'll need to acquire some obsolete e-waste and run it yourself. Something like <a href="https://icop-shop.com/product/ebox-3352dx3-gl/" rel="noopener noreferrer">this</a> based on the Vortex86 chipset looks like a good choice.</p>

<p>There are only two interesting bits of the workflow. The first implements enough of Github's <code>actions/download-artifact</code> for my purposes. In the YAML file it's all one line because YAML and everything that uses it is hateful, I've tidied it up here:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>curl -L -H "X-GitHub-Api-Version: 2022-11-28"
        -H "Authorization: Bearer ${GH_TOKEN}"
        -o dist-for-install.zip
  $(curl -L -s
         -H "Accept: application/vnd.github+json"
         -H "Authorization: Bearer ${GH_TOKEN}"
         -H "X-GitHub-Api-Version: 2022-11-28"
         https://api.github.com/repos/${GH_REPOSITORY}/actions/artifacts
         -o -
    |jq -r '
         [.artifacts.[]|select(.expired == false)]
         |max_by(.created_at).archive_download_url
    '
  )
</code></pre>

</div>



<p>The inner <code>curl</code> uses the <a href="https://docs.github.com/en/rest/actions/artifacts?apiVersion=2022-11-28#list-artifacts-for-a-repository" rel="noopener noreferrer">Github API</a> to fetch a list of all the build artifacts that have ever been created for this repository, as JSON, which it filters using <code>jq</code> to find the most recent one which hasn't expired, and then extract its <code>archive_download_url</code> field. The outer <code>curl</code> then fetches that.</p>

<p>The second bit interesting bit is specific to testing my perl code. Perl can be built on 32-bit platforms to support either 32 or 64 bit integers. On 64-bit machines perl integers are always 64-bit - and there's been a note in the docs for 20 years about how a build-time parameter for 32-bit ints may be added later! I need to test with 32 bit ints, but most OS packages, even on 32-bit machines, will provide a perl with 64-bit ints. So I need to build my own. This is dead simple if a little wordy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>curl -o perl-5.42.0-src.tar.gz https://cpan.metacpan.org/authors/id/B/BO/BOOK/perl-5.42.0.tar.gz
tar xzf perl-5.42.0-src.tar.gz
cd perl-5.42.0
sh Configure -de -Dprefix=$HOME/perl-5.42.0-installed
make -j $(nproc)
make install
$HOME/perl-5.42.0-installed/bin/perl -MCPAN -e 'install qw(App::cpanminus)'
</code></pre>

</div>



<p>The call to the <code>Configure</code> script is where the OS packaging people make it have 64-bit ints by adding the <code>-Duse64bitint</code> flag. Normally you would <code>make test</code> too but I've satisfied myself that it does indeed pass its tests using this Docker image, so I skip that step to save a lot of time.</p>

<p>Finally, I can extract my software and test it. It is bloody irritating that Github automatically wrap my tarball in the obsolete <code>zip</code> format. I blame Microsoft:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>unzip dist-for-install.zip
mkdir dist-for-test
tar -C dist-for-test -xzf *.tar.gz
cd dist-for-test/*
$HOME/perl-5.42.0-installed/bin/cpanm --installdeps .
$HOME/perl-5.42.0-installed/bin/perl Makefile.PL
make test TEST_VERBOSE=1
</code></pre>

</div>



<p>And there you have it - how to test your code automatically, using Github actions, on a 32-bit platform.</p>

