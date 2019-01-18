# vue-player
audio player components based vue.js、iview2.x
# usage

## 1、install
npm install fly-vue-player --save
## 2、example based vue.js iview2.x
``` javascript
<template>
<div>
  <Card>
    <vue-player
      :url="audioUrl"></vue-player>
  </Card>
</div>
</template>


<script>

import VuePlayer from 'fly-vue-player';

export default {
components: {  VuePlayer },
data() {

  return {
    audioUrl: 'https://xxxx.com/mp3/3.mp3'
  }
},
methods: {

}
}
</script>
```
