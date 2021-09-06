<template>
    <div  style="padding:20px 0;border:1px solid red;">
        {{item.index}}--{{item.name}}
    </div>
</template>
<script>
export default {
    props :['item'],
    data() {
        return {}
    },
}
</script>
<style lang='scss' scoped>
</style>