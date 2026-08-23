---
layout: page
title: Search
permalink: /search/
---

<div id="search-container">
  <form role="search" aria-label="Site search" onsubmit="return false">
    <div class="search-field">
      <input type="search" id="search-input" placeholder="Search posts, topics, categories…" autocomplete="off" spellcheck="false" aria-label="Search posts" autofocus>
      <button type="button" id="search-clear" aria-label="Clear search" hidden>×</button>
    </div>
  </form>
  <p id="search-count" aria-live="polite" class="search-count"></p>
  <ul id="results-container" aria-live="polite"></ul>
  <div id="search-empty" hidden>
    <p>No results for “<span id="search-query"></span>”.</p>
    <p class="search-suggestions">Try <a href="{{ site.baseurl }}/categories/">browsing categories</a> or search for <em>Kubernetes</em>, <em>MCP</em>, <em>Generative AI</em>.</p>
  </div>
  <div id="search-initial">
    <p class="search-hint">Type to search posts and reading digests. Try <a href="?q=Kubernetes">Kubernetes</a>, <a href="?q=MCP">MCP</a>, <a href="?q=DeepSeek">DeepSeek</a>.</p>
  </div>
</div>

<script src="{{ site.baseurl }}/assets/simple-jekyll-search.min.js" type="text/javascript"></script>

<script>
(function() {
  var input = document.getElementById('search-input');
  var clearBtn = document.getElementById('search-clear');
  var countEl = document.getElementById('search-count');
  var resultsContainer = document.getElementById('results-container');
  var emptyEl = document.getElementById('search-empty');
  var queryEl = document.getElementById('search-query');
  var initialEl = document.getElementById('search-initial');

  function getQuery() { return input.value.trim(); }

  function escapeRegExp(s) { return s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); }

  function highlight(text, query) {
    if (!query || !text) return text;
    var terms = query.split(/\s+/).filter(Boolean).map(escapeRegExp);
    if (!terms.length) return text;
    var re = new RegExp('(' + terms.join('|') + ')', 'gi');
    return text.replace(re, '<mark class="search-highlight">$1</mark>');
  }

  function updateChrome() {
    var q = getQuery();
    clearBtn.hidden = !q;
    if (!q) {
      countEl.textContent = '';
      emptyEl.hidden = true;
      initialEl.hidden = false;
      return;
    }
    initialEl.hidden = true;
    // After SimpleJekyllSearch renders
    setTimeout(function() {
      var items = resultsContainer.querySelectorAll('.search-result');
      var n = items.length;
      if (n === 0) {
        countEl.textContent = '';
        emptyEl.hidden = false;
        queryEl.textContent = q;
      } else {
        emptyEl.hidden = true;
        countEl.textContent = n + ' result' + (n !== 1 ? 's' : '') + ' for “' + q + '”';
      }
    }, 0);
  }

  // URL sync — read on load
  var params = new URLSearchParams(window.location.search);
  var initialQ = params.get('q');
  if (initialQ) input.value = initialQ;

  SimpleJekyllSearch({
    searchInput: input,
    resultsContainer: resultsContainer,
    json: '{{ site.baseurl }}/search.json',
    searchResultTemplate: '<li class="search-result"><article class="search-card"><a href="{url}" class="search-card-link"><h3 class="search-card-title">{title}</h3></a><p class="search-meta"><span class="search-date">{date}</span><span class="search-category">{category}</span></p><p class="search-excerpt">{excerpt}</p></article></li>',
    noResultsText: '',
    limit: 20,
    fuzzy: true,
    templateMiddleware: function(prop, value, template) {
      var q = getQuery();
      if ((prop === 'title' || prop === 'excerpt' || prop === 'category') && q && value) {
        return highlight(value, q);
      }
      if (prop === 'excerpt' && !value) return '';
      if (prop === 'category' && !value) return '';
      return value;
    }
  });

  // Sync URL and chrome on input
  input.addEventListener('input', function() {
    var q = getQuery();
    var url = new URL(window.location.href);
    if (q) url.searchParams.set('q', q);
    else url.searchParams.delete('q');
    history.replaceState(null, '', url);
    updateChrome();
  });

  clearBtn.addEventListener('click', function() {
    input.value = '';
    input.focus();
    resultsContainer.innerHTML = '';
    var url = new URL(window.location.href);
    url.searchParams.delete('q');
    history.replaceState(null, '', url);
    updateChrome();
  });

  document.addEventListener('keydown', function(e) {
    if (e.key === '/' && document.activeElement !== input && !e.ctrlKey && !e.metaKey && !e.altKey) {
      e.preventDefault();
      input.focus();
    }
    if (e.key === 'Escape' && document.activeElement === input && getQuery()) {
      clearBtn.click();
    }
  });

  // Initial chrome state
  updateChrome();
  // If loaded with ?q=, trigger search (SimpleJekyllSearch listens to input, so dispatch)
  if (initialQ) {
    // Ensure library has loaded json before triggering
    setTimeout(function() {
      input.dispatchEvent(new Event('input', { bubbles: true }));
      // Fallback: if library didn't re-run on programmatic value, force update
      setTimeout(updateChrome, 300);
    }, 200);
  }

  // Also watch for mutations to keep count in sync (library replaces innerHTML)
  var observer = new MutationObserver(updateChrome);
  observer.observe(resultsContainer, { childList: true });
})();
</script>
