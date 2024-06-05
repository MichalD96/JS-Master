
**Code creator: [Michal__d](https://github.com/MichalD96)**

You can use/modify this code if (read [license](https://github.com/MichalD96/JS-Master/blob/master/LICENSE)) and add link to this [repository](https://github.com/MichalD96/JS-Master) above the code you used.

<hr>
<br>

## **Calculate average array of arrays:**

This script allows you to calculate average value of all first, second, third... elements of given arrays.

```javascript
function averageArray(...args) {
  const longestArray = args.reduce((acc, array) =>
    array.length > acc ? array.length : acc, 0
  );

  return new Array(longestArray)
    .fill()
    .map((value, index) =>
      args
        .map(array => array[index] || 0)
        .reduce((acc, val) => acc + val, 0)
    )
    .map(element => element / args.length);
}
```

How to use:

```javascript
const arrays = {
  a: [1, 2, 3, 4, 5, 6],
  b: [10, 20, 30, 40, 50, 60],
  c: [100, 200, 300, 400, 500, 600, 900],
}

averageArray(...Object.values(arrays)); // returns: [37, 74, 111, 148, 185, 222, 300];
```

<hr>
<br>

## **Dynamically generate YouTube video link:**

This class has been made to generate YT video links or HTML code (a elements).

You need only video ID. You can find video ID in link search query, preceded by `?v=` and ends before `&`.


```javascript
// array of objects to create links
// name must be one word, it will be used as object key
// `linkProperties` - object with custom properties for the link
// (`customLinkName` - innerText of `<a>` element, `class` - array of classes added to `<a>` element)
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

Assign every video to the variable so as not to refer every time to the class

`const dragons = Video.videos['dragons']; `

`Video` is the class, `videos` is object of all videos,

`['dragons']` is the name of particular video (you can also use:

`Video.videos.dragons` but first method is safer)

`dragons.getLink(0, 5);` returns only video URL starting from 0 min and 5 seconds

`dragons.getHTML(0, 5);` returns HTML a element with href= video url starting from 0 min and 5 seconds

Append created element on the site:

`document.querySelector('body').append(dragons.getHTML());`

If you not provide arguments to getHTML or getLink methods, video will start from the beginning.



<hr>
<br>

## **Nesting multiple functions in order of execution context:**

Inspired by [Overment](https://www.youtube.com/channel/UC_MIaHmSkt9JHNZfQ_gUmrg)

```javascript
const executeInOrder = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);
```

(assume we have already created functions one, two, three, four) result of this expression:

`four(three(two(one(10))));`

is equal to:

`executeInOrder(one, two, three, four)(10);`

In this case value 10 is provided as an argument to the function one,

The result of function one is provided to function two....

If you prefer to provide arguments in reverse order you can use `.reduceRight()` method.

The main advantage of that method is readability of the code.

`(...fns)` is the set of callback functions that will be executed from left to right.

`x` is our first argument provided to function `one`

`fns.reduce((acc, fn) => fn(acc), x)` method `.reduce()` execute every function and store the final value in `acc`,

`acc` takes the value of result of last executed function,

`fn` is currently executing function,

`fn(acc)` execute callback with last function result as an argument,

`x` is a value provided to .reduce() method as a start value,

### **Similar version for composition of async functions**

```javascript
const executeInOrderAsync = (...fns) => x => fns.reduce((acc, fn) =>
  acc.then(fn), Promise.resolve(x));
```

every function `(...fns)` must be resolved to start executing next async function

<hr>
<br>

## **Advanced data loader:**

Execute fetch of all website data, store it until the rest of the code will be loaded and ready to use it
```javascript
class Loader {
  promises = Object.create(null);
  urls = new Map();
  fetchProperties = { method: 'GET', cache: 'no-cache' };

  static fetchData(urls) {
    urls.forEach(url => {
      const fileName = url.split('/').filter(e => e).pop().split('.', 1)[0];
      this.promises[fileName] = new Loader(url);
      this.urls.set(filename, url);
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
    this.promises[name] = new Loader(this.urls.get(name), { cache: Date.now() });
  }

  static set fetchOptions(obj) {
    this.fetchProperties = Object.assign(this.fetchProperties, obj);
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
    eventsArray.forEach(captureEvent => {
      this.addEventListener(captureEvent, event => {
        callback(event);
      }, config);
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

`config` must be an object same as for regular `.addEventListener()`
  - ex. `{ capture: true, passive: false, once: false }` - all values are set to false as default.
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
    newObj[name] = typeof newObj[name] === 'undefined'
      ? objectDeepCopy(oldObj[name], null)
      : newObj[name];
  }

  return newObj;
}

```
Unfortunately there is no way to copy `getter` and `setter` from an object,

but that is the only limitation.

<hr>
<br>

## **Get search query [key, value] pairs.<br>Generate search quey string from specific localStorage values**

Query string is convenient way to set some site properties or share site settings to other ppl.

```javascript
const SearchQuery = (settingsPrefix => {
  return {
    getParameter(name, url = window.location.search) {
      const obj = Object.create(null);
      new URLSearchParams(url).forEach((value, key) =>
        obj[key] = value ? JSON.parse(value) : '');

      return obj[name] || null;
    },
    getUrl() {
      const { protocol, hostname, pathname, port } = location;
      const url = `${protocol}//${hostname}${port ? ':' + port : ''}${pathname}`;
      const obj = Object.create(null);
      Object.entries(localStorage).forEach(([key, value]) => {
        if (key.startsWith(settingsPrefix))
          obj[key.replace(settingsPrefix, '')] = JSON.parse(value);
      });
      const search = Object.entries(obj)
        .map(pair => pair.map(encodeURIComponent).join('='))
        .join('&');

      return `${url}?${search}`;
    }
  }
})('main.');
```

If you set prefix `"main."` before every localStorage key you want to include to search query

function `SearchQuery.getUrl()` return full link to the site with all values from localStorage.

`SearchQuery.getParameter(<name> [, location]);` returns value of that property


<hr>
<br>

## **Linear proportion with cut values out of range**
`value` - number to convert,

`iMin` - input range minimum,

`iMax` - input range maximum,

`oMin` - output range minimum,

`oMax` - output range maximum;

```javascript
function linear(value, iMin, iMax, oMin, oMax) {
  const clamp = (num, min, max) =>
    num <= min ? min : num >= max ? max : num;
  return clamp((((value - iMin) / (iMax - iMin)) * (oMax - oMin) + oMin), oMin, oMax);
}
```
ex.
`linear(4, 0, 10, 0, 100);`   4 is 40% of range 0 to 10 so final value is 40% of range 0 to 100 = 40

`linear(12, 0, 10, 0, 100);`  12 is 120% of range 0 to 10 so final value is 120% of range 0 to 100 = 120 BUT:

it clamps the final value to output range maximum = 100

<hr>
<br>

## **Easily convert string to 12/24 hours clock format (browser only)**

```javascript
function convertToTime(hours = '00', minutes = '00', format12 = navigator.hour12) {
  minutes = minutes.toString().padStart(2, '0');
  return (format12 || false) ?
    `${hours}:${minutes}` :
    `${+hours % 12 || 12}:${minutes} ${+hours < 12 ? 'AM' : 'PM'}`;
}

function convertToTime(hours = '00', minutes = '00', format12 = navigator.hour12) {
  return new Date(`1970-01-01T${hours}:${minutes.toString().padStart(2, '0')}:00.000Z`)
    .toLocaleTimeString({},
      { timeZone: 'UTC', hour12: format12 , hour: 'numeric', minute: 'numeric' }
    );
}
```
In both cases `convertToTime(16, 45)` returns "16:45" or "4:45",

however first method has much better performance.

It begins noticeable on less efficient PCs,

if you need to create over 1k clocks on the site (yes, it happens sometimes xD)

<hr>
<br>

## **Old padding - `.padStart()` and `.padEnd()` combined in one method:**

`.padStart()` and `.padEnd()` does not support negative numbers, my script has no problem with it

```javascript
for (const Obj of [Number, String]) {
  Object.defineProperty(Obj.prototype, 'padding', {
    value: function (s = 15, x = 5) {
      const number = +this;
      const num = Math.abs(1e15 + ((x > 0)
        ? Math.floor(number)
        : Math.round(number))).toString().slice(-s);
      const dec = (x >= 0) ? (number % 1).toFixed(x).slice(1) : '';
      return `${(number < 0) ? '-' : ''}${num}${dec}`;
    },
  });
};
```
`(16.35).padding(4, 7)` = "0016.3500000"

the limitation is 15 integers, decimal places are limited to standard JS precision = 17

so you should never go over `.padding(15, 17);`

also script can add only '0' on the beginning and on the end, but who needs anything else?


<hr>
<br>

## **The real modulus operator in javaScript:**

In javascript exists only reminder operator, but is easy to write your own modulus operator

```javascript
Object.defineProperty(Number.prototype, 'mod', {
  value: function (number) {
    return ((this % number) + number) % number;
  }
});
```

reminder:

`-2 % 7` = -2

modulus:

`(-2).mod(7)` = 5

<hr>
<br>

## **Calculate centerpoint and radius of circle around set of points / polygon**

`points` - array of objects that must contain "x" and "y" properties.
  - ex. `const positions = [{ "x": 16, "y": 25 }, { "x": 146, "y": 78 }];`

`extendBounds` - amount of units to extend circle radius.

```javascript
function pointsProperties(points, precision = 4) {
  if (!points.length) return;

  const center = (points => {
    return points
      .reduce(([oX, oY], { x, y }) => [[...oX, x], [...oY, y]], [[], []])
      .map(arr => arr.reduce((acc, amount) => acc + amount, 0) / points.length);
  })(points);

  const maxDist = points.reduce((acc, { x, y }) => {
    const [a, b] = [center[0] - x, center[1] - y];
    return Math.max(acc, Math.hypot(a, b));
  }, 0);

  return {
    radius: (extendBounds = 0) => (maxDist + extendBounds).toFixed(precision),
    massCenter: () => center,
  };
}
```

How to use:

`const myPoints = pointsProperties(positions);`

`myPoints.massCenter();` - returns center of the circle (center of the circle is in geometrical mass center of the figure made from connecting all points).

`myPoints.radius(2);` - returns the radius of circle containing all points (extended by 2 units).


<hr>
<br>

## **Reversed `.slice()`**

`.edges(x, y);`

Returns `x` elements from the beginning of the array and `y` elements from the end of the array.
```javascript
Object.defineProperty(Array.prototype, "edges", {
  value: function (a, b) {
    if (a < 0 || b < 0 || (a + b >= this.length))
      return this;

    const right = a ? [...this].reverse().slice(-a) : [];
    const left = b ? [...this].slice(-b).reverse() : [];

    return [...left, ...right].reverse();
  },
});


const arr = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.edges();     // []
arr.edges(3);    // [0, 1, 2]
arr.edges(0, 1); // [9]
arr.edges(0, 3); // [7, 8, 9]
arr.edges(2, 2); // [0, 1, 8, 9]
arr.edges(11);   // value bigger than arr.length - returns original array

```

<hr>
<br>

## **Find extreme values of the array without sorting**

```javascript
Object.defineProperty(Array.prototype, "minMax", {
  value: function () {
    return [Math.min(...this), Math.max(...this)];
  },
});
```
returns array with minimum and maximum value

<hr>
<br>

## **Replace object keys**

```javascript
function replaceObjectKeys(sourceObject, newKeysObject) {
  return Object.keys(sourceObject).reduce((acc, value) => {
    return { ...acc, ...{ [newKeysObject[value] || value]: sourceObject[value] } };
  }, {});
};

const sourceObject = {
  name: 'Michal',
  age: 25,
}

const newKeysObject = {
  name: 'nick',
  age: 'lvl'
};

replaceObjectKeys(sourceObject, newKeysObject) // returns { nick: 'Michal', lvl: 25 }
```

Function returns new object without modifying original object.


<hr>
<br>

## **Cubic Bézier curve**

`point1` - start point coordinates [x, y] of the curve.

`tangentP1` - bezier handler connected to `point1` [x, y].

`point2` - end point coordinates [x, y] of the curve.

`tangentP2` - bezier handler connected to `point2` [x, y].

`complexity` - define how many points, function creates between start and end point of the curve. (keep this value as low as possible)

```javascript
function cubicBezierPoints(point1, tangentP1, point2, tangentP2, complexity = 100) {
  return new Array(complexity)
    .fill([[], []])
    .map((element, index) => {
      const t = index / (complexity - 1);
      return element.map((e, _index) =>
        (1 - t) ** 3 * point1[_index] + 3 * (1 - t) ** 2 * t * tangentP1[_index] + 3 * (1 - t) * t ** 2 * tangentP2[_index] + t ** 3 * point2[_index]);
    });
}
```

Function returns array containing arrays of points to create the curve,

returned array length is equal to complexity parameter,

every element of the array is another array containing one point coordinates [x, y].

<hr>
<br>

## **Simple digits grouping**

```javascript
function groupDigits(string, separator = '.') {
  return string.toString().replace(/\B(?=(\d{3})+(?!\d))/gm, separator);
}
```

```javascript
for (const Obj of [Number, String]) {
  Object.defineProperty(Obj.prototype, 'groupDigits', {
    value: function (separator = '.') {
      return this.toString().replace(/\B(?=(\d{3})+(?!\d))/gm, separator);
    },
  });
}
```

`groupDigits(1234567);`

`(1234567).groupDigits();`

both returns "1.234.567"

For more complex grouping i recommend `.toLocaleString()` method.


<hr>
<br>

## **Recursive comparing two objects**

`compareObjects(obj1, obj2)`

`obj1` is reference object that should contain all properties we are looking for to compare.

```javascript
function compareObjects(obj1, obj2) {
  const wrongProperties = [];
  (function recursiveCompare(obj1, obj2) {
    for (const prop in obj1) {
      if (typeof obj1[prop] === 'object' && obj1[prop] && obj2[prop]) {
        recursiveCompare(obj1[prop], obj2[prop])
      }
      else if (obj1.hasOwnProperty(prop)) {
        if (obj1[prop] !== obj2[prop])
          wrongProperties.push(prop);
      }
    }
  })(obj1, obj2);
  return wrongProperties;
}
```

Function returns array with keys of all not matching properties.

Function can also compare two arrays but it will return indexes instead of object keys.

<hr>
<br>

## **Extract file name and extension from the path**

Easily get file name and extension from the path/link/API

Sometimes working with file system is necessary to dynamically operate on file names and extensions.
This simple method allows you to extract everything you need.

```javascript
Object.defineProperty(String.prototype, 'getFromPath', {
  value: function (type) {
    const path = this.replace(/\\/g, '/');
    const props = {
      name: path.split('/').filter(e => e).pop().split('.', 1)[0],
      extension: path.match(/(?:\.([^.]+))?$/)[1],
      fullPath: path,
      foldersPath: path.substring(0, path.lastIndexOf('/')),
    };
    return props[type in props ? type : 'fullPath'];
  }
});
```

```javascript
const path = './assets/javaScript/getNameProperty.min.js';

// returns 'getNameProperty'
path.getFromPath('name');

// returns 'js'
path.getFromPath('extension');

// returns './assets/javaScript'
path.getFromPath('foldersPath');

// returns full path and converts backslashes to slashes
path.getFromPath('fullPath');
path.getFromPath();
```

<hr>
<br>

## **Find index or snap to the most similar element in array of digits, to given value**

Method looking for the smallest difference between value passed as argument and every element of the array.

The result is index of element with the smallest difference to provided value.

```javascript
Object.defineProperties(Array.prototype, {
  'indexOfClosestValue': {
    value: function (value) {
      return this
        .map(element => Math.abs(value - element))
        .reduce((acc, diff, index, arr) => diff < arr[acc] ? index : acc, 0);
    }
  },
  'snapToClosestValue': {
    value: function (value) {
      const index = this
        .map(element => Math.abs(value - element))
        .reduce((acc, diff, index, arr) => diff < arr[acc] ? index : acc, 0);

      return this[index];
    }
  }
});
```

Example:
```javascript
const array = [-20, Math.PI, 20.1235, Math.log(17), 73, 50, 60, 4, -82, Math.sqrt(2)];

array.indexOfClosestValue(62.45); // returns 6 (array[6] = 60)
array.indexOfClosestValue(3.14); // returns 1 (Math.PI = 3.1415926)

array.snapToClosestValue(62.45); // returns 60
array.snapToClosestValue(2.8); // returns Math.log(17) = 2.833213344056216
```

It is not approximated value by any rounding, script counts absolute smallest difference between given value and array elements.

<hr>
<br>

## **Recursively asynchronously look for all files in the directory and optionally filter them in callback function or extensions array (Node.js)**

Required npm modules: fs-extra, path

```javascript
/**
@param { String } directory - ABSOLUTE path of the directory (always must ends with slash!)
@param { Array } extensions - (optional) extensions of the files you looking for (left undefined to get all types of files)
@param { Array } excludedFolders - (optional) folders names that will be excluded form searching
@param { Function } customFilter - (optional) callback to filter files (name - file name, path - absolute file path, ext - file extension)
@returns { Object Promise => Array }
**/

function getAllFilesAsync({
  directory,
  extensions = [],
  excludedFolders = [],
  customFilter = () => true,
}) {
  return (async function getFiles(directory) {
    const entries = await fs.readdir(directory, { withFileTypes: true });
    const files = entries
      .filter((file) => !file.isDirectory())
      .map((file) => ({ ...file, path: path.join(directory, file.name), ext: path.extname(file.name) }));

    const folders = entries.filter((folder) => folder.isDirectory() && !excludedFolders.includes(folder.name));

    for (const folder of folders) {
      files.push(...await getFiles(`${directory}${folder.name}/`));
    }

    return files
      .filter(({ ext }) => extensions.length ? extensions.some(extension => ext.endsWith(extension)) : true)
      .filter(customFilter);
  })(directory);
};
```

How to use:

```javascript
getAllFilesAsync({
  directory: path.join(__dirname, 'javaScript/'),
  extensions: ['.js', '.mjs'],
  excludedFolders: ['test_files'],
  customFilter: ({ path, name, ext }) => /^[A-Z]{1}[0-9_\w]+\.[a-z]{2,3}$/.test(name);
})
  .then((files) => {
    files.forEach((file) => console.log(file));
  })
  .catch(console.error);
```

`file` is an object containing file name (with extension), absolute file path and file extension.
`customFilter` callback allows you for advanced files filtering

<hr>
<br>

## **Trigger function at specific time**

`UTCtimeString` - time in format 'hours:minutes:seconds' (in UTC/GMT timezone)

`callback` - function to execute

`args` - arguments you pass to the callback function

```javascript
function triggerAt(UTCtimeString, callback, ...args) {
  const [hours = 0, minutes = 0, seconds = 0] = UTCtimeString.split(':').map(Number);

  setTimeout(callback, (() => {
    const now = new Date();
    const ms = new Date(Date.UTC(
      now.getUTCFullYear(),
      now.getUTCMonth(),
      now.getUTCDate(),
      hours,
      minutes,
      seconds,
    )).valueOf() - now.valueOf();

    return ms > 0 ? ms : ms + 864e5;
  })(), ...args);
}
```

```javascript
triggerAt('15:26:00' console.log, "it's 15:26 Universal coordinated time");
```

<hr>
<br>

## **Divide array in two arrays matching, and not matching given filter**

`.partition()` method works very similar to `.filter()` method.

```javascript
Object.defineProperty(Array.prototype, 'partition', {
  value: function(callback) {
    return this.reduce((acc, value, index, array) => {
      if (!!callback(value, index, array))
        return [acc[0].concat(value), acc[1]];

      return [acc[0], acc[1].concat(value)];
    }, [[], []]);
  },
});
```

How to use:
```javascript
const [matching, notMatching] = [1, 45, 100, 17, 60, 12, 51].partition((value) => value < 50);

matching      // value: [1, 45, 17, 12]
notMatching   // value: [100, 60, 51]
```

`callback(current value[, index[, array]])` - function to filter the array, must return true/false, otherwise its result will be converted to boolean value.


<hr>
<br>

## **Interpolate smooth transition between number array values**

`transition` function takes array of values to create animation,

`animation` callback function interpolate values between adjacent elements base on passed value of transition completion.

0% transition is always 0 array element and 100% transition is always last array element.


```javascript
function transition(valuesArray) {
  return (animationCompletion) => {
    const frame = linear(animationCompletion, 0, 1, 0, valuesArray.length - 1);
    const frameCompletion = frame % 1;
    const valueStart = valuesArray[Math.floor(frame)];
    const valueEnd = valuesArray[Math.ceil(frame)];
    return valueStart + (valueEnd - valueStart) * frameCompletion;
  }
}

```

Helper function (described [here](https://github.com/MichalD96/JS-Master#linear-proportion-with-cut-values-out-of-range)):
```javascript
function linear(value, iMin, iMax, oMin, oMax) {
  const clamp = (num, min, max) => num <= min ? min : num >= max ? max : num;
  return clamp((((value - iMin) / (iMax - iMin)) * (oMax - oMin) + oMin), oMin, oMax);
}
```

How to use:

```javascript
const array = [1, 10, 100, 1000, 10000, 100000];

const animation = transition(array);

// animation function as argument accepts value between 0 and 1 which corresponds to the completion state of the transition.
animation(0.5);   // returns 550, 50% of the animation is between 100 and 1000 so (100 + 1000) / 2 = 550
animation(0);     // returns 1, 0% of the animation is 0 array element
animation(0.32);  // returns 64, 0.32 is in 60% transition between 10 and 100
```

<hr>
<br>

## **Get relative time difference string in any locale**


`start` and `end` are set as default to current time

`format` allows you to customize output string

`locale` sets string output language


Uses npm [timediff](https://www.npmjs.com/package/timediff) package


```javascript
import timediff from 'timediff';

const getTimeString = ({
  start = 'now',
  end = 'now',
  format = 'YMDHmS',
  locale = 'en-US',
}) => {
  const order = ['years', 'months', 'days', 'hours', 'minutes', 'seconds'];
  const f = new Intl.RelativeTimeFormat(locale, { style: 'long', numeric: 'always' });
  const difference = timediff(start, end, format);
  const string = Object.entries(difference)
    .sort((...args) => {
      const [a, b] = args.map(key => order.indexOf(key[0]));
      return a - b;
    })
    .filter(([, time]) => time)
    .map(([unit, time]) => {
      const arr = f.formatToParts(time, unit);
      const digit = arr.find(({ type }) => type === 'integer').value;
      const timeUnit = arr.find(({ type, value }) => type === 'literal' && value.startsWith(' ')).value.split(' ')[1];
      return `${digit} ${timeUnit}`;
    })
    .join(', ');

  return string;
};

getTimeString({ end: new Date('2023-01-01'), locale: 'en-US' });    // returns "4 days, 8 hours, 25 minutes, 18 seconds"
getTimeString({ end: new Date('2023-01-01'), locale: 'pl-PL' });    // returns "4 dni, 8 godzin, 25 minut, 18 sekund"
```

<hr>
<br>

## **Parse date and time**

```javascript
import moment from 'moment-timezone';

const parseDate = (date = new Date()) => {
  if (!(date instanceof Date) && !isNaN(Number(date))) {
    date = new Date(Number(date));
  }
  const timezoneString = Intl.DateTimeFormat().resolvedOptions().timeZone;
  const momentDate = moment.tz(date, timezoneString);

  return {
    week: momentDate.isoWeek(),
    now: momentDate.toDate(),
    day: momentDate.startOf('day').toDate(),
    monday: momentDate.clone().isoWeekday(1).startOf('day').toDate(),
    sunday: momentDate.clone().isoWeekday(7).startOf('day').toDate(),
    monthStart: momentDate.clone().startOf('month').toDate(),
    monthEnd: momentDate.clone().endOf('month').toDate(),
    yearStart: momentDate.clone().startOf('year').toDate(),
    yearEnd: momentDate.clone().endOf('year').toDate(),
    addDays(days) {
      const newDate = momentDate.clone().add(days, 'days').toDate();
      return parseDate(newDate);
    },
  };
};

// How to use:
const result = parseDate(new Date());
console.log(result);
console.log('Add 5 days:', result.addDays(5));
console.log('Subtract 3 days:', result.addDays(-3));

```