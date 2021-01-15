
**Code creator: [Michal__d](https://github.com/MichalD96)**

You can use/modify this code if (read [license](https://github.com/MichalD96/JS-Master/blob/master/LICENSE)) and add link to this [repository](https://github.com/MichalD96/JS-Master) above the code you used.

<hr>
<br>

## **Calculate average array of arrays:**

This script allows you to calculate average value of all first, second, third... elements of given arrays.

```javascript
const arrays = {
  a: [1, 2, 3, 4, 5, 6],
  b: [10, 20, 30, 40, 50, 60],
  c: [100, 200, 300, 400, 500, 600, 900],
}

function averageArray(...args) {
  // find the longest array
  const lArray = args.reduce((acc, value, index, array) =>
    array[acc].length < value.length ? index : acc, 0);

  return args[lArray]
    // map values of each array on the longest array
    .map((value, index) => {
      return args
        // if array element is undefined assign 0 else assign element with given index
        .map(array => array[index] || 0)
        // reduce array of all same index elements from all arrays to one number
        .reduce((acc, val) => acc + val);
    })
    // lArray is array of sum of all elements, divide every element by amount of arguments to get average value
    .map(element => element / args.length);
}

averageArray(...Object.values(arrays)); // function returns: [37, 74, 111, 148, 185, 222, 300];
```

<hr>
<br>

## **Dynamically generate YouTube video link:**

This class has been made to generate YT video links or HTML code (a elements).
You need only video ID. You can find video ID in link search query, preceded by `?v=` and ends before `&`.


```javascript
// array of objects to create links
// name must be one word, it will be used as object key
// `linkProperties` - object with custom properties for the link (`customLinkName` - innerText of `<a>` element, `class` - array of classes added to `<a>` element)
const videosCollection = [
  { name: 'broken', videoId: 'oAscVnqeF00', linkProperties: { customLinkName: 'Broken', class: ['yt-video-link', 'vid'] } },
  { name: 'dragons', videoId: '7wtfhZwyrcc' },
];


class Video {
  constructor(videoId, linkProperties = {}) {
    Object.assign(this, Object.assign({ customLinkName: 'Video', class: ['yt-video-link'] }, linkProperties));
    this.videoId = videoId;
  }
  static init(videosCollection) {
    this.videos = {};
    videosCollection.forEach(({ name, videoId, linkProperties }) => {
      this.videos[name] = new Video(videoId, linkProperties);
    });
  }
  getLink(...timeData) {
    const [min = 0, sec = min ? 0 : 1] = timeData;
    return `https://www.youtube.com/watch?v=${this.videoId}&t=${min * 60 + sec}s`;
  }
  getHTML(...timeData) {
    const link = document.createElement('a');
    link.innerText = this.customLinkName;
    link.classList.add(...this.class);
    link.setAttribute('href', this.getLink(...timeData));
    link.setAttribute('target', '_blank');
    return link;
  }
};

Video.init(videosCollection);
```
**How to use:**

```javascript
// assign every video to the variable so as not to refer every time to the class
const dragons = Video.videos['dragons']; // `Video` is the class, `videos` is object of all videos,
// ['dragons'] is the name of particular video (you can also use: `Video.videos.dragons` but first method is safer)

dragons.getLink(0, 5); // returns only video URL starting from 0 min and 5 seconds
dragons.getHTML(0, 5); // returns HTML a element with href= video url starting from 0 min and 5 seconds

// append created element on the site:
document.querySelector('body').append(dragons.getHTML());  // if you not provide arguments to getHTML or getLink methods, video will start from the beginning.

```

<hr>
<br>

## **Nesting multiple functions in order of execution context:**

```javascript
const executeInOrder = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

// (assume we have already created functions one, two, three, four) result of this expression:
four(three(two(one(10))));
// is equal to:
executeInOrder(one, two, three, four)(10);
// in this case value 10 is provided as an argument to the function one, the result of function one is provided to function two....
```
If you prefer to provide arguments in reverse order you can use `.reduceRight()` method.

The main advantage of that method is readability of the code.

`(...fns)` is the set of callback functions that will be executed from left to right.

`x` is our first argument provided to function `one`

`fns.reduce((acc, fn) => fn(acc), x)` method `.reduce()` execute every function and store the final value in `acc`,

`acc` takes the value of result of last executed function,

`fn` is currently executing function,

`fn(acc)` execute callback with last function result as an argument,

`x` is a value provided to .reduce() method as a start value,


<hr>
<br>

## **Advanced data loader:**

Execute fetch of all website data, store it until the rest of the code will be loaded and ready to use it
```javascript
class Loader {
  static promises = {};
  static urls = new Set();
  static fetchProperties = { method: 'GET', cache: 'no-cache' };

  static fetchData(urls) {
    urls.forEach(url => {
      this.urls.add(url);
      const fileName = this.getName(url);

      if (!this.promises[fileName])
        this.promises[fileName] = new Loader(url);
      else
        throw new Error(`"${fileName}" already registered!`);
    });

    this.contentLoaded = new Promise(resolve => this.resolveContentLoaded = resolve);
  }
  constructor(url, noCache = null) {
    this._json = (async () => {
      try {
        const response = await fetch(url, noCache || Loader.fetchProperties);
        return await response.json();
      } catch (err) {
        throw new Error(`Failed to load: ${url}\n${err}`);
      }
    })();
  }
  execute(...args) {
    const json = this._json;
    delete this._json;
    return json.then(...args);
  }
  static reloadData(name) {
    delete this.promises[name];
    const url = this.urls.find(url => this.getName(url) === name);
    this.promises[name] = new Loader(url, { cache: 'no-cache' });
  }
  static set fetchOptions(obj) {
    this.fetchProperties = Object.assign(this.fetchProperties, obj);
  }
  static getName(url) {
    return url.split('/').pop().split('.', 1)[0];
  }
}

const urlsSetOne = [
  'https://jsonplaceholder.typicode.com/users',
];
const urlsSetTwo = [
  'https://jsonplaceholder.typicode.com/posts',
  'https://jsonplaceholder.typicode.com/albums',
];

Loader.fetchOptions = { method: 'GET', cache: 'no-cache' };
Loader.fetchData(urlsSetOne); // Execute this run fetch of all data

Loader.fetchOptions = { method: 'POST' };
Loader.fetchData(urlsSetTwo);


// Loader.promises returns new Promise then assigning it to variable
// allows to use it with Promise.all and confirm that everything has been loaded
const f1 = Loader.promises['users'].execute(data => { console.log(data) });
const f2 = Loader.promises['posts'].execute(data => { console.log(data) });
const f3 = Loader.promises['albums'].execute(data => { console.log(data) });

Promise.all([f1, f2, f3]).then(Loader.resolveContentLoaded);

Loader.contentLoaded.then(() => { console.log('All data loaded') });

// you can reload data by:
Loader.reloadData('users');
```
<hr>
<br>

## **Custom Super Event Listener:**

This eventListener listen multiple events on one element.
It base on the same syntax as regular `addEventListener()`
`element.addListener(events array, callback[, configuration object*])`

```javascript
Object.defineProperty(EventTarget.prototype, 'addListener', {
  value: function (eventsArray, callback, config) {
    const configObject = Object.assign({ capture: false, passive: false, once: false }, config);
    eventsArray.forEach(captureEvent => {
      this.addEventListener(captureEvent, event => {
        callback(event);
      }, configObject);
    });
  },
  enumerable: false,
  configurable: false,
  writable: false,
});

document.querySelector('button')
  .addListener(['contextmenu', 'click', 'mouseover', 'mousedown'], event => {
    event.preventDefault();
    console.log(event);
    event.stopPropagation();
  }, { capture: true });
```

<hr>
<br>

## **Recursive deep copying of an object:**

```javascript
function objectDeepCopy(oldObj = {}, newObj) {
  if (oldObj === null || typeof oldObj !== 'object' || ![Object, Array].includes(oldObj.constructor))
    return oldObj;
  if ([Date, RegExp, Function, String, Number, Boolean].includes(oldObj.constructor))
    return new oldObj.constructor(oldObj);

  newObj = newObj || new oldObj.constructor();

  for (const name in oldObj) {
    newObj[name] = typeof newObj[name] === 'undefined' ? objectDeepCopy(oldObj[name], null) : newObj[name];
  }

  return newObj;
}

```
Unfortunately there is no way to copy `getter` and `setter` from an object,
but that is the only limitation.