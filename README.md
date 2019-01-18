# vue-player
audio player components based vue.js、iview2.x
# usage
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

import {VuePlayer} from 'zyzl-components';

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
