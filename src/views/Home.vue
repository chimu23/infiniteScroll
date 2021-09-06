<template>
  <div class="home">
    <scroll :remain="8" :size="30" :items="list" :variable="true">
      <template #items="{item}">
          <item :item="item"></item>
      </template>
    </scroll>
  </div>
</template>

<script>
// @ is an alias to /src
import scroll from '@/components/scroll.vue'
import item from '@/components/item.vue'
import mock from 'mockjs'
export default {
  name: 'Home',
  components: {
    scroll,
    item
  },
  data() {
    const list = Array(300)
      .fill({})
      .map((item, index) => {
        return { name: mock.Random.sentence(5,30), index }
      })
    return {
      list,
   
    }
  },
  created() {},
}
</script>
