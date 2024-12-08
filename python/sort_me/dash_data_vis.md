<!DOCTYPE html>
<html lang="en"><head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1"><!-- Begin Jekyll SEO tag v2.8.0 -->
<title>Data Visualization With Dash | My References</title>
<meta name="generator" content="Jekyll v3.9.2" />
<meta property="og:title" content="Data Visualization With Dash" />
<meta property="og:locale" content="en_US" />
<meta name="description" content="Write an awesome description for your new site here. You can edit this line in _config.yml. It will appear in your document head meta (for Google search results) and in your feed.xml site description." />
<meta property="og:description" content="Write an awesome description for your new site here. You can edit this line in _config.yml. It will appear in your document head meta (for Google search results) and in your feed.xml site description." />
<link rel="canonical" href="http://0.0.0.0:4000/python/sort_me/dash_data_vis.md" />
<meta property="og:url" content="http://0.0.0.0:4000/python/sort_me/dash_data_vis.md" />
<meta property="og:site_name" content="My References" />
<meta property="og:type" content="website" />
<meta name="twitter:card" content="summary" />
<meta property="twitter:title" content="Data Visualization With Dash" />
<script type="application/ld+json">
{"@context":"https://schema.org","@type":"WebPage","description":"Write an awesome description for your new site here. You can edit this line in _config.yml. It will appear in your document head meta (for Google search results) and in your feed.xml site description.","headline":"Data Visualization With Dash","url":"http://0.0.0.0:4000/python/sort_me/dash_data_vis.md"}</script>
<!-- End Jekyll SEO tag -->
<link rel="stylesheet" href="/assets/main.css"><link type="application/atom+xml" rel="alternate" href="http://0.0.0.0:4000/feed.xml" title="My References" /></head>
<body><header class="site-header" role="banner">

  <div class="wrapper"><a class="site-title" rel="author" href="/">My References</a><nav class="site-nav">
        <input type="checkbox" id="nav-trigger" class="nav-trigger" />
        <label for="nav-trigger">
          <span class="menu-icon">
            <svg viewBox="0 0 18 15" width="18px" height="15px">
              <path d="M18,1.484c0,0.82-0.665,1.484-1.484,1.484H1.484C0.665,2.969,0,2.304,0,1.484l0,0C0,0.665,0.665,0,1.484,0 h15.032C17.335,0,18,0.665,18,1.484L18,1.484z M18,7.516C18,8.335,17.335,9,16.516,9H1.484C0.665,9,0,8.335,0,7.516l0,0 c0-0.82,0.665-1.484,1.484-1.484h15.032C17.335,6.031,18,6.696,18,7.516L18,7.516z M18,13.516C18,14.335,17.335,15,16.516,15H1.484 C0.665,15,0,14.335,0,13.516l0,0c0-0.82,0.665-1.483,1.484-1.483h15.032C17.335,12.031,18,12.695,18,13.516L18,13.516z"/>
            </svg>
          </span>
        </label>

        <div class="trigger"></div>
      </nav></div>
</header>
<main class="page-content" aria-label="Content">
      <div class="wrapper">
        <article class="post">

  <header class="post-header">
    <h1 class="post-title">Data Visualization With Dash</h1>
  </header>

  <div class="post-content">
    <h2 id="what-is-dash">What is Dash</h2>

<p>Dash is an open sources framework for building data visualization interfaces. Dash uses Flask to create a web server, React.js to render the user interface and Plotly.js to generate charts. Dash takes care of making these technologies work together.</p>

<h2 id="install">Install</h2>

<p>Naturally you want to do this in a virtual environment.</p>

<p>You can install Dash with pip using the command <code class="language-plaintext highlighter-rouge">pip install dash==2.0.0</code> (you don’t have to pin the version just an example)</p>

<p>In the lesson we also installed Pandas so the full command was <code class="language-plaintext highlighter-rouge">pip install dash==2.0.0 pandas==1.3.3</code></p>

<p>To actually get it running I had to downgrade <code class="language-plaintext highlighter-rouge">numpy</code> to version <code class="language-plaintext highlighter-rouge">1.26.4</code> and run the latest version of <code class="language-plaintext highlighter-rouge">dash</code> <code class="language-plaintext highlighter-rouge">2.18.1</code> and the above <code class="language-plaintext highlighter-rouge">pandas</code> <code class="language-plaintext highlighter-rouge">1.3.3</code></p>

<h2 id="building-your-app">Building your app</h2>

<p>Its kind of tricky to note the content so this is an example from the course to get you started. <a href="https://github.com/xcomfan/real_python_examples/tree/main/dash_intro">avocado statistics dahs example</a></p>

<h2 id="running-your-app-in-dev-mode">Running your app in dev mode</h2>

<p>With the virtual environment sourced use the command <code class="language-plaintext highlighter-rouge">python app.py</code> (assuming app.py is your entry point) to launch your app in development mode. When running in the development mode changes made will be automatically reloaded.</p>

<h2 id="deploying">Deploying</h2>

<p>Heroku is one option that the class covers, but use the docs and make it work for you if needed.</p>

  </div>

</article>

      </div>
    </main><footer class="site-footer h-card">
  <data class="u-url" href="/"></data>

  <div class="wrapper">

    <h2 class="footer-heading">My References</h2>

    <div class="footer-col-wrapper">
      <div class="footer-col footer-col-1">
        <ul class="contact-list">
          <li class="p-name">My References</li><li><a class="u-email" href="mailto:your-email@example.com">your-email@example.com</a></li></ul>
      </div>

      <div class="footer-col footer-col-2"><ul class="social-media-list"><li><a href="https://github.com/jekyll"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg> <span class="username">jekyll</span></a></li><li><a href="https://www.twitter.com/jekyllrb"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#twitter"></use></svg> <span class="username">jekyllrb</span></a></li></ul>
</div>

      <div class="footer-col footer-col-3">
        <p>Write an awesome description for your new site here. You can edit this line in _config.yml. It will appear in your document head meta (for Google search results) and in your feed.xml site description.</p>
      </div>
    </div>

  </div>

</footer>
</body>

</html>
