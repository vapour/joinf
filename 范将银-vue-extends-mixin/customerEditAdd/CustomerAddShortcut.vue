<script>
import CustomerEditAddBase from './CustomerAdd'
import customerApi from 'api/customerApi'
import { mapGetters } from 'vuex'
export default {
    name: 'CustomerAddShortcut',
    extends: CustomerEditAddBase,
    data() {
        return {
            type: 3,
            /**
             * 快捷状态
             */
            shortcutStatus: true
        }
    },
    computed: {
        ...mapGetters({
            navTags: 'navTag/getNavTags',
        })
    },
    methods: {
        saveFun() {
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
                    this.$message.success(this.langs.newSuccess)
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
            this.$store.dispatch('navTag/addNavTag', {
                tagId: 'customerAdd',
                name: 'customerAdd',
                tagName: this.langs.newCustomer,
                closable: true,
                targetId: 'customerAdd',
                params: {
                    operate: 'add',
                    tagId: 'customerAdd',
                    currentTab: this.currentTab,
                    ruleForm: this.getModelsData(this.ruleForm),
                    CTRuleForm: this.getContactModelsData(this.CTRuleForm)
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
