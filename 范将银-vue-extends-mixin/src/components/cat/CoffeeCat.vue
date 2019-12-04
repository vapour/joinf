<script>
import CatBase from '@/components/cat/CatBase'
import say from '@/components/say/spark'

export default {
    name: 'CoffeeCat',
    extends: CatBase,
    mixins: [say],
    data() {
        return {
            name: '咖啡猫',
            color: 'Coffee'
        }
    }
}
</script>

<style>

</style>
