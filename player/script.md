
<div id="atour-article-player" dir="rtl" aria-label="آموزش ویدئویی سئو">
  <div class="atv-media">
    <video muted playsinline preload="none"
           aria-label="ویدئوی آموزش سئو"></video>

    <div class="atv-controls">
      <button type="button" class="atv-play">پخش</button>
      <span class="atv-count"></span>
      <button type="button" class="atv-sound"
              aria-pressed="false">روشن کردن صدا</button>
      <button type="button" class="atv-next">بعدی</button>
    </div>
  </div>

  <div class="atv-copy">
    <span class="atv-badge">آموزش ویدئویی سئو</span>
    <strong>سئو را کاربردی یاد بگیرید</strong>
    <p>قدم‌به‌قدم با اصول بهینه‌سازی سایت آشنا شوید.</p>
    <a href="https://atourtech.ir/contact-us/"
       target="_blank" rel="noopener sponsored">
      مشاوره سئو با آتورتک
    </a>
    <span class="atv-status" role="status" aria-live="polite"></span>
  </div>
</div>

<style>
#atour-article-player,
#atour-article-player * {
  box-sizing: border-box;
}

#atour-article-player {
  width: 100%;
  max-width: 620px;
  margin: 24px 0 24px auto;
  display: flex;
  overflow: hidden;
  border: 1px solid #dce6ee;
  border-radius: 14px;
  background: #fff;
  color: #173c6c;
  font-family: inherit;
  text-align: right;
}

#atour-article-player .atv-media {
  position: relative;
  flex: 0 0 48%;
  min-width: 0;
  background: #101820;
}

#atour-article-player video {
  display: block;
  width: 100%;
  height: 100%;
  min-height: 180px;
  max-height: 230px;
  object-fit: contain;
  background: #101820;
}

#atour-article-player .atv-controls {
  position: absolute;
  inset: auto 0 0;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 5px;
  padding: 8px 5px;
  color: #fff;
  background: linear-gradient(transparent, rgba(0,0,0,.88));
}

#atour-article-player button {
  min-height: 34px;
  margin: 0;
  padding: 5px 7px;
  border: 1px solid rgba(255,255,255,.35);
  border-radius: 6px;
  background: rgba(0,0,0,.45);
  color: #fff;
  font: inherit;
  font-size: 11px;
  cursor: pointer;
}

#atour-article-player .atv-count {
  font-size: 11px;
  white-space: nowrap;
}

#atour-article-player .atv-copy {
  flex: 1;
  min-width: 0;
  padding: 16px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 8px;
}

#atour-article-player .atv-badge {
  color: #007f8c;
  font-size: 11px;
}

#atour-article-player strong {
  color: #173c6c;
  font-size: 17px;
  line-height: 1.7;
}

#atour-article-player p {
  margin: 0;
  font-size: 13px;
  line-height: 1.9;
}

#atour-article-player .atv-copy a {
  display: block;
  padding: 9px 8px;
  border-radius: 8px;
  background: #00a9b7;
  color: #fff;
  font-size: 13px;
  line-height: 1.8;
  text-align: center;
  text-decoration: none;
}

#atour-article-player .atv-status {
  font-size: 11px;
  line-height: 1.7;
}

#atour-article-player .atv-status:empty {
  display: none;
}

#atour-article-player :focus-visible {
  outline: 3px solid #f2b544;
  outline-offset: 2px;
}

@media (max-width: 480px) {
  #atour-article-player .atv-copy {
    padding: 10px;
    gap: 6px;
  }

  #atour-article-player strong {
    font-size: 14px;
  }

  #atour-article-player p {
    font-size: 12px;
  }

  #atour-article-player video {
    min-height: 200px;
  }
}
</style>

<script>
(() => {
  'use strict';

  const root = document.getElementById('atour-article-player');
  if (!root || root.dataset.ready) return;
  root.dataset.ready = '1';

  const urls = [
    'https://operators.atourmovies.ir/movies/seo/7be7084b9dd0f70d089af595ec846d55/hls/master.m3u8',
    'https://operators.atourmovies.ir/movies/seo/3398aad448566a8d1ce58ccfac950684/hls/master.m3u8',
    'https://operators.atourmovies.ir/movies/seo/e0de4412723ea2943afb613668a3ed58/hls/master.m3u8',
    'https://operators.atourmovies.ir/movies/seo/cedad83c42e688608c31ae29e491aeea/hls/master.m3u8',
    'https://operators.atourmovies.ir/movies/seo/9781c77b61846c7390d24a6cd3b6801b/hls/master.m3u8'
  ];

  const video = root.querySelector('video');
  const playButton = root.querySelector('.atv-play');
  const soundButton = root.querySelector('.atv-sound');
  const nextButton = root.querySelector('.atv-next');
  const counter = root.querySelector('.atv-count');
  const status = root.querySelector('.atv-status');

  let index = 0;
  let hls = null;
  let started = false;
  let userPaused = false;
  let generation = 0;
  let timer = null;
  let hlsPromise = null;

  const failed = new Set();

  video.muted = true;
  video.defaultMuted = true;

  function clearTimer() {
    clearTimeout(timer);
    timer = null;
  }

  function controls() {
    playButton.textContent = video.paused ? 'پخش' : 'توقف';
    soundButton.textContent =
      video.muted ? 'روشن کردن صدا' : 'قطع صدا';
    soundButton.setAttribute('aria-pressed', String(!video.muted));
  }

  function getHls() {
    if (window.Hls) return Promise.resolve(window.Hls);
    if (hlsPromise) return hlsPromise;

    hlsPromise = new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = 'https://cdn.jsdelivr.net/npm/hls.js@1/dist/hls.min.js';
      script.onload = () => window.Hls
        ? resolve(window.Hls)
        : reject(new Error('HLS unavailable'));
      script.onerror = () => reject(new Error('HLS load failed'));
      document.head.appendChild(script);
    });

    return hlsPromise;
  }

  async function play() {
    if (userPaused) return;
    const token = generation;

    try {
      await video.play();
    } catch (error) {
      if (token !== generation || userPaused) return;

      if (error.name === 'NotSupportedError') {
        fail();
      } else if (error.name !== 'AbortError') {
        clearTimer();
        status.textContent = 'برای شروع، دکمه پخش را بزنید.';
      }
    }

    controls();
  }

  function armTimeout() {
    clearTimer();
    const token = generation;

    timer = setTimeout(() => {
      if (token === generation && !userPaused) fail();
    }, 30000);
  }

  function fail() {
    if (userPaused) return;
    failed.add(index);
    next();
  }

  function next() {
    for (let step = 1; step <= urls.length; step++) {
      const candidate = (index + step) % urls.length;

      if (!failed.has(candidate)) {
        load(candidate);
        return;
      }
    }

    clearTimer();
    generation++;
    userPaused = true;
    video.pause();
    if (hls) hls.stopLoad();
    status.textContent = 'پخش ممکن نیست؛ برای تلاش دوباره پخش را بزنید.';
    controls();
  }

  async function load(nextIndex) {
    clearTimer();
    const token = ++generation;
    index = nextIndex;
    started = true;

    if (hls) {
      hls.destroy();
      hls = null;
    }

    video.pause();
    video.removeAttribute('src');
    video.load();

    counter.textContent =
      (index + 1).toLocaleString('fa-IR') +
      ' / ' + urls.length.toLocaleString('fa-IR');

    status.textContent = 'در حال بارگذاری…';
    armTimeout();

    if (video.canPlayType('application/vnd.apple.mpegurl')) {
      video.src = urls[index];
      video.load();
      play();
      return;
    }

    try {
      const Hls = await getHls();
      if (token !== generation) return;

      if (!Hls.isSupported()) {
        clearTimer();
        userPaused = true;
        status.textContent = 'مرورگر شما از پخش این ویدئو پشتیبانی نمی‌کند.';
        controls();
        return;
      }

      hls = new Hls();
      hls.on(Hls.Events.MANIFEST_PARSED, () => {
        if (token === generation) play();
      });

      hls.on(Hls.Events.ERROR, (event, data) => {
        if (token === generation && data.fatal) fail();
      });

      hls.loadSource(urls[index]);
      hls.attachMedia(video);
    } catch (error) {
      if (token !== generation) return;
      clearTimer();
      userPaused = true;
      status.textContent = 'بارگذاری پلیر ممکن نشد؛ صفحه را دوباره باز کنید.';
      controls();
    }
  }

  video.addEventListener('playing', () => {
    clearTimer();
    status.textContent = '';
    controls();
  });

  video.addEventListener('pause', controls);
  video.addEventListener('volumechange', controls);

  video.addEventListener('ended', () => {
    if (!userPaused) next();
  });

  video.addEventListener('error', () => {
    if (started && !userPaused && video.error) fail();
  });

  video.addEventListener('waiting', () => {
    if (userPaused) return;
    status.textContent = 'در حال بارگذاری…';
    armTimeout();
  });

  playButton.addEventListener('click', () => {
    if (!started) {
      userPaused = false;
      load(0);
    } else if (!video.paused) {
      userPaused = true;
      clearTimer();
      video.pause();
      status.textContent = 'پخش متوقف است.';
    } else {
      userPaused = false;

      if (failed.size === urls.length) {
        failed.clear();
        load(0);
      } else {
        if (hls) hls.startLoad();
        armTimeout();
        play();
      }
    }

    controls();
  });

  soundButton.addEventListener('click', () => {
    video.muted = !video.muted;
    controls();
  });

  nextButton.addEventListener('click', () => {
    userPaused = false;
    if (failed.size === urls.length) failed.clear();
    next();
  });

  if ('IntersectionObserver' in window) {
    const observer = new IntersectionObserver(entries => {
      if (!entries.some(entry => entry.isIntersecting)) return;
      observer.disconnect();
      if (!started) load(0);
    }, { threshold: 0.15 });

    observer.observe(root);
  }

  window.addEventListener('pagehide', () => {
    clearTimer();
    video.pause();
    if (hls) hls.stopLoad();
  });

  window.addEventListener('pageshow', event => {
    if (event.persisted && started && !userPaused) {
      if (hls) hls.startLoad();
      play();
    }
  });

  controls();
})();
</script>
