<template>
    <div>
        <div class="box">
            <h1>404 NotFound</h1>
        </div>
        <div class="back">
            <router-link to="/">
            <button>返回首页</button>
            </router-link>
            <!-- <a href="#/">也能返回</a>
            一般使用上面那种 -->
        </div>
    </div>
</template>

<script lang="ts">
    export default {
        name: 'NotFound'
    }
</script>

<style scoped="scss">
.box{
    position: relative;
}
h1{
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    font-size: 70px;
    margin-top: 50px;
}
.back{
    position: relative;
}
button{
    position: absolute;
    background-color: rgb(219, 219, 219);
    width: 100px;
    height: 40px;
    font-size: 15px;
    border:0px black solid;
    border-radius: 5px;
    top: 180px;
    left: 50%;
    transform: translateX(-50%);
    color: rgb(0, 0, 0);
}
button:hover{
    box-shadow: 1px 1px 1px rgb(41, 41, 41);
}

</style>