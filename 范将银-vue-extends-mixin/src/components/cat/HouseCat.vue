<script>
import CatBase from '@/components/cat/CatBase'
import say from '@/components/say/spark'
import phone from '@/components/say/phone'
export default {
    name: 'HouseCat',
    extends: CatBase,
    mixins: [say, phone],
    provide() {
        return {
            name: this.name,
            param: this.body
        }
    },
    data() {
        return {
            name: '家猫',
        }
    },
    mounted() {
        this.body.limbs = '穿鞋'
        this.body.hair = '穿衣'
        this.body.lang = '主人教我唱歌、说话'
    },
    methods: {
        say() {
            this.call()
        }
    }
}
</script>

<style>

</style>
