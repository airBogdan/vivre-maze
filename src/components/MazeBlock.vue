<template>
    <div class="maze">
<!--        {{column}}-->
    </div>

</template>

<script>
    export default {
        props: ['column']
    }
</script>

<style scoped>
    .maze {
        height: 100px;
        width: 100px;
        border: 1px solid lightblue;
        box-sizing: border-box;
    }
</style>
