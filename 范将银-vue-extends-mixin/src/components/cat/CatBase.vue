<template>
    <el-card>
        <el-card class="box_card">
            <div slot="header" class="clearfix">
                <span class="ti">{{name}}</span>
                <el-button style="float: right; padding: 3px 0" type="text" @click="say">开始喵星语</el-button>
            </div>
            <div v-for="(o, key) in body" :key="o" class="text item">
                <span>{{`${key.toUpperCase()}: ${o}`}}<span v-if="color && key === 'hair'">&#8195;&#8195;&#8195;&#8195;{{color}}</span></span>
            </div>
            <photo v-if="name === '家猫'" class="photo"></photo>
        </el-card>
    </el-card>
</template>

<script>
import say from '@/components/say/cat'
import photo from '@/components/photo'
export default {
    name: 'CatBase',
    components: {photo},
    mixins: [say],
    data() {
        return {
            name: '猫',
            body: {
                eyes: '眼睛',
                mouth: '嘴巴',
                limbs: '四肢',
                hair: '皮毛',
                lang: '瞄瞄!!!'
            },
            color: null
        }
    }
}
</script>

<style lang="scss" scoped>
    .text {
        font-size: 14px;
    }
    .item {
        margin-bottom: 18px;
    }
    .clearfix{
        .ti{
            font-size: 24px;
            font-weight: bold;
        }
        &:before,
        &:after {
            display: table;
            content: "";
        }
        &:after {
            clear: both
        }
    }
    .box_card {
        position: relative;
        width: 480px;
        .photo{
            position: absolute;
            right: 20px;
            top: 80px;
        }
    }
</style>
