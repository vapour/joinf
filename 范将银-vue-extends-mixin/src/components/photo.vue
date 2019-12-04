<template>
    <div class="photoVue">
        <div class="ti">{{name}}</div>
        <div>{{param.lang}}</div>
    </div>
</template>

<script>
export default {
    name: 'photo',
    inject: {
        name: {
            type: String,
            default: ''
        },
        param: {
            type: Object,
            default: {}
        }
    },
    mounted() {
        this.$nextTick(()=> {
            this.param.hair = '爱洗澡，穿新衣'
            this.name = '本猫'
        })
    }
}
</script>

<style lang='scss' scoped>
    .photoVue{
        width: 80px;
        height: 120px;
        background-color: #eeeeee;
        font-size: 12px;
        .ti {
            font-size: 18px;
            font-weight: 500;
        }
    }
</style>
