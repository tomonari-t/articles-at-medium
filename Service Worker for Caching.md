- Service Worker
- App Shell Model

# Service Worker for Caching

Caching by Service Worker is good for Web App because we don't have user to wait for page showing. It make user feel using native apps.

So at this post I'll introduce to how to cache data by using Service Worker.

## What is "App Shell"

![App Shell Image](https://developers.google.com/web/fundamentals/architecture/images/appshell.png?hl=ja)

"App Shell" is design of Web Application.

"App Shell" means that minimum HTML, CSS, JavaScript to construct Web App. If you make your Web-App cache "App Shell's resources" at user's local, user can only load other contents when visiting again without loading "App Shell".

"App Shell" architecture is good for Single-Page-Applications.

So caching minimum HTML, CSS, JavaScript by Service Worker is the important key to realize it.

## How to use Service Worker for Caching

You need only to do two things.

1. Register SW
2. Add Event listner func

※ All code is in this [repository | GitHub](https://github.com/tomonari-t/app-shell-demo).

### 1. Register Service Worker

To install a service worker, in your JavaScript you have to register it.

```JS
<html>
...
<script>
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('./sw.js').then((registration) => {
    console.log('ServiceWorker registration successful with scope: ', registration.scope);
  }).catch((err) => {
    console.log('ServiceWorker registration failed: ', err);
  });
}
</script>
...
</html>
```

### 2. Add Event listners func

After registering service worker, you can add envent listners to cache files.

At `install` event, you can tell browser what files should be cached. After installed, `install` event is not called until new cache files appear.

```JS
const cacheName = 'app-shell-demo.v1.0.0';
const fileToCacheList = [
  '/mdl.css',
  '/style.css',
  'index.html',
  '/'
];

const addCache = (event) => {
  event.waitUntil((async () => {
    const cache = await caches.open(cacheName);
    return cache.addAll(fileToCacheList);
  })());
};
self.addEventListener('install', addCache);
```

At `activate` event, you had better remove old cached files.

```JS
// sw.js
const removeOldCache =  (event) => {
  const cacheWhiteList = [cacheName];
  event.waitUntil((async () => {
    const keys = await caches.keys();
    Promise.all(
      keys.map((key) => {
        if (cacheWhiteList.indexOf(key) === -1) {
          return caches.delete(key);
        }
      })
    );
  })());
};
self.addEventListener('activate',removeOldCache);
```

At `fetch` event is called at every requests. So you can return chached file at this event. If there is no data at cache, you have to fetch the data.

```JS
self.addEventListener('fetch', returnCacheIfExist);
const returnCacheIfExist = (event) => {
  event.respondWith((async () => {
    const matched = await caches.match(event.request, { ignoreSearch: true });
    return matched || fetch(event.request); // if nothing, fetch
  })());  
};

```

## More detail

See [this demo app. | iFixit](https://github.com/GoogleChromeLabs/sw-precache/tree/master/app-shell-demo)

It is good example by `React`. 

## Wrapping up

"App Shell" provides good experience for users.

## Reference

- [App Shell Model](https://developers.google.com/web/fundamentals/architecture/app-shell)

---

Portions of this page are reproduced from work created and shared by Google and used according to terms described in the Creative Commons 3.0 Attribution License.