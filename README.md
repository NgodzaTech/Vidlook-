<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>VidFind - Zimbabwe Videos</title>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #000; color: #fff; }
    header { background: red; padding: 10px 15px; font-size: 20px; font-weight: bold; display: flex; justify-content: space-between; align-items: center; }
    nav a { color: white; text-decoration: none; margin-left: 15px; }
    #shortsRow { display: flex; overflow-x: auto; background: #111; padding: 10px; }
    .short-thumb { width: 120px; height: 200px; margin-right: 10px; border-radius: 10px; object-fit: cover; cursor: pointer; }
    .video-box { background: #111; margin: 10px; padding: 10px; border-radius: 8px; }
    .video-box img { width: 100%; border-radius: 8px; cursor: pointer; }
    button { background: red; color: white; padding: 6px 10px; border: none; border-radius: 4px; margin-top: 8px; cursor: pointer; }
    #fullScreenShorts { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: black; z-index: 9999; display: none; overflow-y: scroll; scroll-snap-type: y mandatory; }
    .short-slide { height: 100vh; scroll-snap-align: start; display: flex; align-items: center; justify-content: center; }
    .short-slide iframe { width: 100%; height: 100%; border: none; }
    #closeFull { position: absolute; top: 10px; right: 20px; font-size: 30px; color: white; cursor: pointer; z-index: 1000; }
  </style>
</head>
<body>

<header>
  VidFind
  <nav>
    <a href="#" onclick="loadVideos()">Home</a>
    <a href="#" onclick="alert('Downloads page coming soon')">Downloads</a>
  </nav>
</header>

<div style="padding: 10px;">
  <input type="text" id="searchInput" placeholder="Search Zimbabwean videos" style="width: 70%; padding: 6px;" />
  <button onclick="searchVideos()">Search</button>
</div>

<!-- Horizontal Shorts -->
<div id="shortsRow"></div>

<!-- Main Video Container -->
<div id="videoList"></div>

<!-- Full Screen Short View -->
<div id="fullScreenShorts">
  <span id="closeFull" onclick="closeShorts()">&times;</span>
  <div id="shortSlides"></div>
</div>

<script>
const apiKey = 'AIzaSyDoE4SbBMkn3kFa4M9SvIEz6mgjkuU9a5M';
let nextPageToken = '';
let currentQuery = 'Zimbabwe music';

function loadVideos(query = 'Zimbabwe music', append = false) {
  fetch(`https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&maxResults=10&regionCode=ZW&pageToken=${nextPageToken}&q=${encodeURIComponent(query)}&key=${apiKey}`)
    .then(res => res.json())
    .then(data => {
      nextPageToken = data.nextPageToken || '';
      const videoList = document.getElementById('videoList');
      if (!append) videoList.innerHTML = '';

      let shortRow = '';
      let shortSlides = '';

      data.items.forEach((item, index) => {
        const id = item.id.videoId;
        const title = item.snippet.title;
        const thumb = item.snippet.thumbnails.medium.url;

        // Main video list
        videoList.innerHTML += `
          <div class="video-box">
            <img src="${thumb}" onclick="openShort(${index})" />
            <p>${title}</p>
            <button onclick="window.open('https://www.y2mate.com/youtube/${id}', '_blank')">Download ↓</button>
          </div>
        `;

        // Horizontal shorts
        if (index < 10 && !append) {
          shortRow += `<img class="short-thumb" src="${thumb}" onclick="openShort(${index})"/>`;
        }

        shortSlides += `
          <div class="short-slide">
            <iframe src="https://www.youtube.com/embed/${id}?autoplay=1&mute=1" allowfullscreen></iframe>
          </div>
        `;
      });

      if (!append) document.getElementById('shortsRow').innerHTML = shortRow;
      document.getElementById('shortSlides').innerHTML = shortSlides;
    });
}

function searchVideos() {
  const input = document.getElementById('searchInput').value.trim();
  if (input) {
    currentQuery = input;
    nextPageToken = '';
    loadVideos(currentQuery);
  }
}

function openShort(index) {
  const viewer = document.getElementById('fullScreenShorts');
  viewer.style.display = 'block';
  viewer.scrollTop = index * window.innerHeight;
}

function closeShorts() {
  document.getElementById('fullScreenShorts').style.display = 'none';
}

function handleScroll() {
  if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight - 500 && nextPageToken) {
    loadVideos(currentQuery, true); // load more and append
  }
}

window.addEventListener('scroll', handleScroll);

// Auto load on page start
window.onload = () => {
  loadVideos();
};
</script>
</body>
</html>const API_KEY = 'AIzaSyDoE4SbBMkn3kFa4M9SvIEz6mgjkuU9a5M';
const searchBox = document.getElementById('searchBox');
const searchBtn = document.getElementById('searchBtn');
const videoContainer = document.getElementById('videoContainer');

let query = 'trending';
let pageToken = '';
let isFetching = false;

async function fetchVideos(newSearch = false) {
  if (isFetching) return;
  isFetching = true;
  const endpoint = `https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&q=${encodeURIComponent(query)}&maxResults=10&pageToken=${pageToken}&key=${API_KEY}`;
  const res = await fetch(endpoint);
  const data = await res.json();
  pageToken = data.nextPageToken || '';
  renderVideos(data.items, newSearch);
  isFetching = false;
}

function renderVideos(videos, clear = false) {
  if (clear) videoContainer.innerHTML = '';
  videos.forEach(video => {
    const videoId = video.id.videoId;
    const wrapper = document.createElement('div');
    wrapper.className = 'video-wrapper';

    const iframe = document.createElement('iframe');
    iframe.src = `https://www.youtube.com/embed/${videoId}?autoplay=1&mute=1&controls=0&playsinline=1&rel=0`;
    iframe.allow = 'autoplay; encrypted-media';

    wrapper.appendChild(iframe);
    videoContainer.appendChild(wrapper);
  });
}

searchBtn.onclick = () => {
  query = searchBox.value || 'trending';
  pageToken = '';
  fetchVideos(true);
};

searchBox.addEventListener('keypress', e => {
  if (e.key === 'Enter') {
    query = searchBox.value || 'trending';
    pageToken = '';
    fetchVideos(true);
  }
});

videoContainer.addEventListener('scroll', () => {
  if (videoContainer.scrollTop + window.innerHeight >= videoContainer.scrollHeight - 500) {
    fetchVideos();
  }
});

window.onload = () => {
  fetchVideos(true);
};self.addEventListener('install', event => {
  console.log('Service Worker installed');
  self.skipWaiting();
});

self.addEventListener('fetch', function(event) {
  event.respondWith(fetch(event.request));
});body {
  margin: 0;
  overflow: hidden;
  background-color: black;
  font-family: Arial, sans-serif;
}

#controls {
  position: fixed;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 5px;
  z-index: 1000;
}

#searchBox {
  padding: 10px;
  font-size: 18px;
  width: 70vw;
}

#searchBtn {
  padding: 10px;
  font-size: 18px;
  cursor: pointer;
}

#videoContainer {
  height: 100vh;
  overflow-y: scroll;
  scroll-snap-type: y mandatory;
}

.video-wrapper {
  height: 100vh;
  scroll-snap-align: start;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

iframe {
  width: 100vw;
  height: 100vh;
  border: none;
}
