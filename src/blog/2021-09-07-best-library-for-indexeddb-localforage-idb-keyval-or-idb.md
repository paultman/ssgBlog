---
title: "Best wrapper for IndexedDB: localForage, IDB-Keyval, or IDB"
date: "2021-09-07"
categories: 
  - "tech"
image: "aadfd-hero.jpg"
---

Continuing from a previous [post](https://paultman.wordpress.com/from-localstorage-to-indexeddb/), moving from Local Storage to IndexedDB, I set about a journey to fully embrace IndexedDB. Not wanting to reinvent the wheel, by enabling promises/async in calls to indexedDB, I've decided to use a 3rd party wrapper.

I started by looking at the top libraries focusing on popularity, if they had a fallback, if they could be used on on the server side (node), if they supported sync with a SS database, and how easy it would be to import/export the raw data.

<table><tbody><tr><td>Library</td><td>Stars/<br>used by</td><td>IndexedDB</td><td>localStorage (fallback)</td><td>NodeJS</td><td>Sync</td><td>Export / Import</td></tr><tr><td><a href="https://dexie.org/" target="_blank" rel="noreferrer noopener">Dexie.js</a></td><td>7k/8k</td><td>Yes</td><td><a href="https://github.com/dfahlander/Dexie.js/issues/480" target="_blank" rel="noreferrer noopener">No,&nbsp;not really.</a></td><td>/= can polyfill</td><td><a rel="noreferrer noopener" href="https://dexie.org/docs/Syncable/Dexie.Syncable.js" target="_blank">/=adapter required</a></td><td><a href="https://dexie.org/docs/ExportImport/dexie-export-import" target="_blank" rel="noreferrer noopener">Yes (adapter)</a></td></tr><tr><td><a href="https://pouchdb.com/" target="_blank" rel="noreferrer noopener">PouchDB</a></td><td>14k/</td><td>Yes</td><td><a href="https://pouchdb.com/adapters.html" target="_blank" rel="noreferrer noopener">Yes (adapter)</a></td><td><a href="https://pouchdb.com/adapters.html" target="_blank" rel="noreferrer noopener">Yes (adapter)</a></td><td>Yes!</td><td><a href="https://github.com/pouchdb-community/pouchdb-load" target="_blank" rel="noreferrer noopener">/ = maybe with some&nbsp;other libs</a></td></tr><tr><td><a href="https://rxdb.info/" target="_blank" rel="noreferrer noopener">RxDB</a></td><td>17k/744</td><td>Yes</td><td>No</td><td>Yes</td><td>Yes</td><td><a href="https://rxdb.info/rx-database.html#dump" target="_blank" rel="noreferrer noopener">Yes</a></td></tr><tr><td><a href="https://github.com/localForage/localForage" target="_blank" rel="noreferrer noopener">localForage</a></td><td>19k/176k</td><td>Yes</td><td>Yes</td><td>No</td><td>No</td><td><a href="https://github.com/localForage/localForage/issues/718" target="_blank" rel="noreferrer noopener">No</a></td></tr><tr><td><a href="https://github.com/typicode/lowdb" target="_blank" rel="noreferrer noopener">lowdb</a></td><td>16k/138k</td><td>No</td><td>Yes</td><td>Yes</td><td>No</td><td>kind of, JSON</td></tr><tr><td><a href="https://github.com/jakearchibald/idb-keyval" target="_blank" rel="noreferrer noopener">idb-keyval</a></td><td>2k/34k</td><td>Yes</td><td>No</td><td>No</td><td>No</td><td>No</td></tr><tr><td><a href="https://github.com/jakearchibald/idb" target="_blank" rel="noreferrer noopener">idb</a></td><td>4k/168k</td><td>Yes</td><td>No</td><td>No</td><td>No</td><td>No</td></tr></tbody></table>

This was a high level evaluation, you can see my more detailed notes in this [Google Spreadsheet](https://docs.google.com/spreadsheets/d/1P-B0FXj4A-ucwqQdmfOH7FV0NVxc3ftr/edit?usp=sharing&ouid=103394891988793194776&rtpof=true&sd=true)

In my particular case, I was looking for a replacement for LocalStorage, so I need it to use IndexDB. I don't care about a fallback due to all major browsers now supporting indexedDB, I would only be using it on the client side, and I didn't care about export/import capabilities.

Being able to sync would have been nice, but my serverside db is MongoDB, which isn't an option with any of the wrappers. I did have a few conversations with Nolan of PouchDB and to be honest, if I wasn't already married to MongoDB and have my own sync algorithm, I'd give it a more serious consideration.

Here's a [graph](https://www.npmtrends.com/dexie-vs-idb-vs-idb-keyval-vs-localforage-vs-pouchdb) from npm trends regarding the popularity for the top packages.

Based on my needs, popularity, and how recently the libraries were updated. I narrowed it down to localForage, IDB, and IDB-Keyval.

LocalForage is the oldest and hence has support for older browsers, and so has support for all 3 types of client storage, (localstorage, websql, and indexeddb). After reviewing the API, I decided to pass on it, as even now, much of their docs mention fallback capabilities; like serializing json to strings for LocalStorage if necessary. Although it's very popular, and used by 176k other projects, I decided to focus on IDB and the Keyval version, both written by Jake Archibald of Google.

IDB came first in 2017, then IDB-Keyval a year later. Why would he make two libraries basically doing the same thing? After reviewing the docs and APIs, it looks like the first version was a promised based api around all the typical indexedDB functions. It's basically indexedDB with "ennhancements." As Jake says, it's "library that mostly mirrors the IndexedDB API, but with small improvements that make a big difference to usability." If you are familiar with using indexedDB manually, the IDB package will feel the same. It does add things like returning promises rather than using IDBRequest event triggers.

IDB-Keyval on the other hand, follows more closely with the LocalStorage API. LocalStorage's ease of use was a key reason for its popularity over the more complex indexedDB. Besides adding promises, IDB didn't address that complexity, so maybe that's why Jake later created IDB-Keyval.

If your use of a local store happens to be limited to key-value storage, it seems like it might be the better version including extra methods like update (rather then get/set), and setMany (rather than using a Promise.All) which aren't in IDB. They both however share the basics like get/set/delete/clear/keys. In the keyvalue examples of IDB, Jake mentioned, "This is very similar to `localStorage`, but async. If this is _all_ you need, you may be interested in [idb-keyval](https://www.npmjs.com/package/idb-keyval). You can always upgrade to this library later."

So, that's what I'm going to do. Start with IDB-Keyval and upgrade IF necessary, rather than using half of a library and possibly having to keep up with newer versions which update code I might never use, not to mention possible future API changes.

My advise is if your use-case calls for full client database functionality like: custom queries, ordering, using cursors with ranges and direction, sorting, etc, go with [IDB](https://github.com/jakearchibald/idb). If, on the other hand, you simply need LocalStorage functionality, use [IDB-Keyval](https://github.com/jakearchibald/idb-keyval). Both are readily updated and, in my view, represent the best two options for indexedDB wrappers.

BTW, if necessary, I will update this article as I'll be implementing the aforementioned library in my own application in the coming days. If you don't see an update, that means things went well and i'm happy with my decision.
