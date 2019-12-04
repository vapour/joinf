<script>
import CustomerEditAddBase from './CustomerEditAddBase'
import customerApi from 'api/customerApi'
import { mapGetters } from 'vuex'

export default {
    name: 'CustomerAdd',
    extends: CustomerEditAddBase,
    data() {
        return {
            type: 0
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
                    console.log(res.success)
                    this.$message.success(this.langs.newSuccess)
                    // 0:保存 1:保存并新建
                    if (this.saveType === 0) {
                        this.$store.dispatch('navTag/replaceNavTag', {
                            tagId: 'customerDetail' + res.data,
                            name: 'customerDetail',
                            tagName: this.customerName,
                            closable: true,
                            targetId: 'customerAdd',
                            params: {
                                customerId: res.data,
                                pageFrom: this.currentTab
                            }
                        })
                    } else {
                        this.ruleForm.forEach((list)=> {
                            if (list.categoryName === '联系人') {
                                if (this.vShow.contactMode === 1) {
                                    // 卡片模式
                                    this.$refs.contactRuleForm[0].resetFields()
                                } else {
                                    // 列表模式
                                    this.$refs.contactRuleTableForm[0].resetFields()
                                }
                            } else {
                                this.$refs[this.ruleFormPrefix + list.refName][0].resetFields()
                            }
                        })
                        this.$refs.elscrollbar.$refs.wrap.scrollTop = 0
                        this.init()
                        this.$nextTick(() => {
                            this.$refs.elscrollbar.update()
                        })
                    }
                } else {
                    this.$message.error(res.errMsg)
                    this.doSaving = false
                }
            })
        }
    }
}
</script>

<style>

</style>
