<script>
import CatBase from '@/components/cat/CatBase'
import say from '@/components/say/en'
export default {
    name: 'PersianCat',
    extends: CatBase,
    mixins: [say],
    data() {
        return {
            name: '波斯猫',
            color: 'black'
        }
    }
}
</script>

<style>

</style>
