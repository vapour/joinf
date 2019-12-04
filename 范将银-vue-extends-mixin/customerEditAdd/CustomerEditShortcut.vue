<script>
import CustomerEditAddBase from './CustomerEdit'
import customerApi from 'api/customerApi'
import { mapGetters } from 'vuex'
export default {
    name: 'CustomerEditShortcut',
    extends: CustomerEditAddBase,
    data() {
        return {
            type: 4
        }
    },
    computed: {
        ...mapGetters({
            navTags: 'navTag/getNavTags',
        })
    },
    methods: {
        saveFun() {
            // 未改动，不提交数据
            if (!this.isAlter) {
                this.$message.success(this.langs.saveSuccess)
                this.saveAndClose()
                return false
            }
            let params = {
                contacts: this.getContactParam(),
                models: this.getModels(),
                customerAttachmentDtoList: this.getAttachments(),
            }
            if (this.getParamValue('from') && this.getParamValue('from') !== '') {
                params.from = this.getParamValue('from')
                if (this.getNavLinkParams(this.navTags, 'fromEmailtype') === ('new' || 'edit')) {
                    params.from = 'email'
                }
            }
            customerApi.postCustomer(params).then(({ data: res })=>{
                this.saveStatus = false
                // console.log('新建客户', res)
                if (res.success) {
                    this.$message.success(this.langs.saveSuccess)
                    this.saveAndClose()
                } else {
                    this.$message.error(res.errMsg)
                    this.doSaving = false
                }
            })
        },
        /**
         * 全屏打开
         */
        fullscreenOpen() {
            this.$store.dispatch('navTag/replaceNavTag', {
                tagId: 'customerEdit' + this.customerId,
                name: 'customerEdit',
                tagName: this.langs.editCustomer,
                closable: true,
                targetId: 'customerDetail' + this.customerId,
                params: {
                    operate: 'edit',
                    tagId: 'customerEdit' + this.customerId,
                    id: this.customerId,
                    pageFrom: this.currentTab,
                    ruleForm: this.ruleForm,
                    CTRuleForm: this.CTRuleForm
                }
            })
            this.cancel()
        },
        /**
         * 取消
         */
        cancel() {
            this.$emit('close')
        },
        close() {
            this.cancel()
        },
        saveAndClose() {
            this.$emit('close')
            this.$emit('update')
        }
    }
}
</script>

<style>

</style>
