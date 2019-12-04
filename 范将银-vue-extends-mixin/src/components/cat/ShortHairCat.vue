<script>
import CatBase from '@/components/cat/CatBase'
export default {
    name: 'ShortHairCat',
    extends: CatBase,
    data() {
        return {
            name: '短尾猫',
            myColor: ['红', '黄', '蓝', '  但是我还是最喜欢这个颜色!']
        }
    },
    methods: {
        say() {
            this.$message.success('我不喜欢说话，但是我会变色')
            this.color = this.myColor[parseInt(Math.random() * 5)]
        }
    }
}
</script>

<style>

</style>
