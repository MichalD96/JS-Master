
**Code creator: [Michal__d](https://github.com/MichalD96)**

You can use/modify this code if you add link to this [repository](https://github.com/MichalD96/JS-Master) over the code you used.

License: MIT

<hr>

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
        // reduce array of all first elements from all arrays to one number
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
// assign every video to the variable so as not to refer every time to the class, ['broken'] is name of particular video
const dragons = Video.videos['dragons']; // `Video` is the class, `videos` is object of all videos, ['dragons'] is the name of particular video (you can also use: `Video.videos.dragons` but first method is safer)

dragons.getLink(0, 5); // returns only video URL starting from 0 min and 5 seconds
dragons.getHTML(0, 5); // returns HTML a element with href= video url starting from 0 min and 5 seconds

// append created element on the site:
document.querySelector('body').append(broken.getHTML());  // if you not provide arguments to getHTML or getLink methods, video will start from the beginning.

```