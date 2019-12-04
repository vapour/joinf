<script>
import CustomerEditAddBase from './CustomerEditAddBase'
import customerApi from 'api/customerApi'

export default {
    name: 'CustomerEdit',
    extends: CustomerEditAddBase,
    data() {
        return {
            type: 1
        }
    },
    methods: {
        saveFun() {
            // 未改动，不提交数据
            if (!this.isAlter) {
                this.saveExit()
                return false
            }
            let params = {
                contacts: this.getContactParam(),
                id: this.customerId,
                models: this.getModels(),
                customerAttachmentDtoList: this.getAttachments(),
            }
            customerApi.patchCustomer(params).then(({ data: res })=>{
                this.saveStatus = false
                console.log(res)
                if (res.success) {
                    this.saveExit()
                } else {
                    this.$message.error(res.errMsg)
                    this.doSaving = false
                }
            })
        },
        saveExit() {
            this.$message.success(this.langs.saveSuccess)
            // 0:保存 1:保存并新建
            if (this.saveType === 0) {
                this.$store.dispatch('navTag/replaceNavTag', {
                    tagId: 'customerDetail' + this.customerId,
                    name: 'customerDetail',
                    tagName: this.customerName,
                    closable: true,
                    targetId: 'customerEdit' + this.customerId,
                    params: {
                        customerId: this.customerId,
                        pageFrom: this.currentTab
                    }
                })
            } else {
                this.$store.dispatch('navTag/replaceNavTag', {
                    tagId: 'customerAdd',
                    name: 'customerAdd',
                    tagName: this.langs.newCustomer,
                    closable: true,
                    targetId: 'customerEdit' + this.customerId,
                    params: {
                        operate: 'add',
                        tagId: 'customerAdd'
                    }
                })
            }
        }
    }
}
</script>

<style>

</style>
