<template>
<div class="customer_edit_add_vue" :class="{shortcut: shortcutStatus}" v-loading="loading">
    <div class="a_s_g_p_bar f14 fc02">
        {{langs.fastNew}}
        <div class="btn">
            <i class="btn_box">
                <icon-hover type="iconxinchuangkoudakai" :size="16" :color="{color: regularColor, hover: primaryIconLightHover, active: primaryIconLightactive}" @click.native="fullscreenOpen"></icon-hover>
            </i>
            <i class="btn_box">
                <icon-hover type="iconguanbi" :size="16" :color="{color: regularColor, hover: primaryIconLightHover, active: primaryIconLightactive}" @click.native="closeConfirm"></icon-hover>
            </i>
        </div>
    </div>
    <el-scrollbar class="x-hidden" ref="elscrollbar" style="height: 100%">
        <dl class="c-content" v-for="(list, pos) in ruleForm" :key="list.categoryName" :ref="list.refName">
            <div v-if="list.categoryName !=='联系人' && (!shortcutStatus || list.domains.length > 0)">
                <dt class="f14 fc02 no-sel pointer" @click="list.expand = !list.expand">
                    {{list.categoryName}}
                    <i class="arrow" :class="{develop: list.expand}" v-show="list.categoryName !=='基础信息'">
                        <icon class="icon" type="iconzhankai4" :size="14" :color="emailListOperateIcon"></icon>
                    </i>
                </dt>
                <dd v-show="list.categoryName ==='基础信息' || list.expand">
                    <div class="base-info t-flex-row">
                        <el-form :model="ruleForm[pos]" :ref="ruleFormPrefix + list.refName" class="demo-ruleForm">
                            <ul class="flex-col-auto">
                                <li :class="isMultiline(domain) ? 'item2 f12 fc02' : 'item1 f12 fc02'" v-for="(domain, index) in ruleForm[pos].domains" :key="domain.id">
                                    <!-- 1/整数;2/小数-金额;3/日期;4/是否;5/图片;6/数据字典;7/客户选择;71/供应商选择;8/用户选择;9/自定义编码;10/字符;11/百分比;12/产品选择  -->
                                    <!-- 客户代码 -->
                                    <el-form-item class="el-code" v-if="domain.id === 'code'" :key="domain.id" :prop="`domains.${index}.originalValue`" :ref="`${list.refName}domains.${index}.originalValue`" :rules="domain.rule">
                                        <span slot="label" :title="domain.columnName">{{domain.columnName}}</span>
                                        <el-input class="JOINF mr24" clearable v-model="domain.originalValue" :maxlength="domain.characterLength" :disabled="domain.force || domain.isReadonly === 1 ? true : false"></el-input>
                                        <i v-if="domain.id === 'code'" class="icon_code iconfont iconerweima f16 a_fc03" @click="getGeneratedCode(ruleForm[pos].domains[index], ruleFormPrefix + list.refName, `domains.${index}.originalValue`)"></i>
                                    </el-form-item>
                                    <!-- 客户名称 -->
                                    <el-form-item v-else-if="domain.id === 'name'" :key="domain.id" :prop="`domains.${index}.originalValue`" :ref="`${list.refName}domains.${index}.originalValue`" :rules="domain.rule">
                                        <span slot="label" :title="domain.columnName">{{domain.columnName}}</span>
                                        <el-input class="JOINF" clearable v-model="domain.originalValue" :maxlength="domain.characterLength" :type="domain.property === 2 ? 'textarea' : 'text'" :rows="1">
                                        </el-input>
                                    </el-form-item>
                                    <!-- 国家/地区 -->
                                    <el-form-item v-else-if="domain.id === 'displayRegion'" :key="domain.id" :prop="`domains.${index}.originalValue`" :ref="`${list.refName}domains.${index}.originalValue`" :rules="domain.rule">
                                        <span slot="label" :title="domain.columnName">{{domain.columnName}}</span>
                                        <dataload-select
                                            ref="countrySelect"
                                            clearable
                                            filterable
                                            showEname
                                            v-model="domain.originalValue"
                                            dataKey="country"
                                            :disabled="domain.isReadonly === 1 ? true : false"
                                            :placeholder="langs.pleaseSelect + domain.columnName">
                                        </dataload-select>
                                    </el-form-item>
                                    <customize-general
                                        v-else
                                        v-model="ruleForm[pos].domains[index]"
                                        :customerId="customerId"
                                        @validateField="validateField(ruleFormPrefix + list.refName, `domains.${index}.originalValue`)"
                                        @changeDisplayType="changeDisplayType"
                                        :prop="`domains.${index}.originalValue`"
                                        :ref="`${list.refName}domains.${index}.originalValue`">
                                    </customize-general>
                                </li>
                            </ul>
                        </el-form>
                    </div>
                </dd>
            </div>
            <div v-else-if="list.categoryName ==='联系人'">
                <dt class="has-icon-right f14 fc02">
                    {{list.categoryName}}
                    <div class="t-switch-btn fr line20 m-t-10">
                        <el-tooltip :enterable="false" :content="langs.cardMode" :open-delay="300" placement="top" effect="dark">
                            <span :class="{'active': vShow.contactMode === 1}" @click="vShow.contactMode = 1">
                                <icon class="icon m-t-2" type="iconkapianshi" :size="14" :color="emailListOperateIcon"></icon>
                            </span>
                        </el-tooltip>
                        <el-tooltip :enterable="false" :content="langs.listMode" :open-delay="300" placement="top" effect="dark">
                            <span :class="{'active': vShow.contactMode === 2}" @click="vShow.contactMode = 2">
                                <icon class="icon m-t-2" type="iconliebiaoshi" :size="14" :color="emailListOperateIcon"></icon>
                            </span>
                        </el-tooltip>
                    </div>
                </dt>
                <dd class="contact-info-content">
                    <div>
                        <div class="tool_bar">
                            <el-checkbox v-show="vShow.contactMode === 1" v-model="vShow.contactAll" class="all_btn" @change="selectAllContact">{{langs.checkAll || '全选'}}</el-checkbox>
                            <div class="b_btn_group">
                                <el-button class="t_btn" @click="delContact(false)">
                                    <icon class="icon m-t-2" type="iconshanchu" :size="18" :color="emailListOperateIcon"></icon>
                                </el-button>
                                <el-button class="t_btn f18" type="primary" icon="el-icon-plus" @click="addContact"></el-button>
                            </div>
                        </div>
                        <!-- 卡片模式——列表 -->
                        <div v-if="vShow.contactMode === 1" class="contact-cards">
                            <div class="card-item ul_box"
                                :class="{expand: item2.expand, hasExpand: item2.num > 6}"
                                v-for="(item2, pos2) in CTRuleForm.contactColumnLists"
                                :key="item2.key">
                                <el-form ref="contactRuleForm" :model="CTRuleForm">
                                    <ul class="contact_item flex-col-auto" :class="{default: item2.defaultContact === 1, checked: item2.checked}" @click="selectContact(item2)">
                                        <li :class="isMultiline(li) ? 'item2 f12 fc02' : 'item1 f12 fc02'" v-show="li.isShow === 1 && (item2.expand || index < 6)" v-for="(li, index) in CTRuleForm.contactColumnLists[pos2].domains" :key="li.lable">
                                            <!-- 性别 -->
                                            <el-form-item
                                            v-if="li.id === 'sex' && li.isShow === 1"
                                            :key="li.key"
                                            :prop="`contactColumnLists.${pos2}.domains.${index}.originalValue`"
                                            :ref="`contactColumnLists.${pos2}.domains.${index}.originalValue`"
                                            :rules="li.rule">
                                                <span slot="label" :title="li.columnName">{{li.columnName}}</span>
                                                <el-radio-group v-model="CTRuleForm.contactColumnLists[pos2].domains[index].originalValue">
                                                    <el-radio
                                                        v-for="MItem in li.dictionarys"
                                                        :label="MItem.value"
                                                        :key="MItem.value">
                                                        {{MItem.chineseName}}
                                                    </el-radio>
                                                </el-radio-group>
                                            </el-form-item>
                                            <!-- 状态 -->
                                            <el-form-item
                                            v-else-if="li.id === 'flag' && li.isShow === 1"
                                            :key="li.key"
                                            :prop="`contactColumnLists.${pos2}.domains.${index}.originalValue`"
                                            :ref="`contactColumnLists.${pos2}.domains.${index}.originalValue`"
                                            :rules="li.rule">
                                                <span slot="label" :title="li.columnName">{{li.columnName}}</span>
                                                <el-switch
                                                    class="small"
                                                    v-model="CTRuleForm.contactColumnLists[pos2].domains[index].originalValue"
                                                    :width="28"
                                                    :active-value="li.dictionarys[0].value"
                                                    :active-text="li.dictionarys[0].chineseName"
                                                    :inactive-value="li.dictionarys[1].value"
                                                    :inactive-text="li.dictionarys[1].chineseName">
                                                </el-switch>
                                            </el-form-item>
                                            <customize-general
                                                v-else-if="li.isShow === 1"
                                                :customerId="customerId"
                                                v-model="CTRuleForm.contactColumnLists[pos2].domains[index]"
                                                :key="li.key"
                                                @validateField="validateField('contactRuleForm', `contactColumnLists.${pos2}.domains.${index}.originalValue`)"
                                                :ref="`contactColumnLists.${pos2}.domains.${index}.originalValue`"
                                                :prop="`contactColumnLists.${pos2}.domains.${index}.originalValue`">
                                            </customize-general>
                                        </li>
                                        <li class="clear display">
                                            <span v-if="item2.expand" class="d_btn f12 fc01 no-sel pointer" @click.stop="collapseConstact(item2)"><icon class="icon m-t-2 develop" type="iconzhankai4" :size="16" :color="primaryIconLight"></icon>{{langs.collapse || '收起'}}</span>
                                            <span v-else class="d_btn f12 fc01 no-sel pointer" @click.stop="expandConstact(pos2)"> <icon class="icon m-t-2" type="iconzhankai4" :size="16" :color="primaryIconLight"></icon>{{langs.expand || '展开'}}</span>
                                        </li>
                                        <!-- 默认联系人标志 -->
                                        <div class="default_btn no-sel pointer" @click="setDefaultContact(pos2)">
                                            <div class="bg"></div>
                                            <div class="text f12" :class="{fcWhite: item2.defaultContact === 1, fc02: item2.defaultContact !== 1}">{{langs.default || '默认'}}</div>
                                        </div>
                                    </ul>
                                </el-form>
                            </div>
                            <span class="clear f0"></span>
                        </div>
                        <!-- 表格模式——列表 -->
                        <div v-else class="contact_table">
                            <el-form ref="contactRuleTableForm" :model="CTRuleForm">
                                <el-table ref="contactTable" :data="CTRuleForm.contactColumnLists" border>
                                    <el-table-column type="selection" width="40" fixed></el-table-column>
                                    <a v-for="(li, index) in contactDefaultForm.domains" :key="li.lable">
                                        <el-table-column
                                            v-if="li.id === 'flag' && li.isShow === 1"
                                            :prop="li.name"
                                            :label="li.columnName"
                                            min-width="160">
                                            <template v-if="li.isRequired === 1" slot="header">
                                                <span><span class="fcErr">*&#8194;</span>{{li.columnName}}</span>
                                            </template>
                                            <template slot-scope="scope">
                                                <el-form-item
                                                    class="inlineMessage"
                                                    :prop="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`"
                                                    :ref="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`"
                                                    :key="scope.row.key"
                                                    :rules="li.rule">
                                                    <el-switch
                                                        class="small"
                                                        v-model="scope.row.domains[index].originalValue"
                                                        :width="28"
                                                        :active-value="li.dictionarys[0].value"
                                                        :active-text="li.dictionarys[0].chineseName"
                                                        :inactive-value="li.dictionarys[1].value"
                                                        :inactive-text="li.dictionarys[1].chineseName">
                                                    </el-switch>
                                                </el-form-item>
                                            </template>
                                        </el-table-column>
                                        <el-table-column
                                            v-if="li.id === 'defaultContact'"
                                            :prop="li.name"
                                            :label="langs.default"
                                            min-width="160">
                                            <template v-if="li.isRequired === 1" slot="header">
                                                <span><span class="fcErr">*&#8194;</span>{{langs.default}}</span>
                                            </template>
                                            <template slot-scope="scope">
                                                <el-form-item
                                                    class="inlineMessage"
                                                    :prop="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`"
                                                    :ref="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`"
                                                    :key="scope.row.key"
                                                    :rules="li.rule">
                                                    <el-switch
                                                        class="small"
                                                        v-model="scope.row.defaultContact"
                                                        @change="setDefaultContact(scope.$index, true)"
                                                        :width="28"
                                                        :active-value="1"
                                                        active-text="是"
                                                        :inactive-value="0"
                                                        inactive-text="否">
                                                    </el-switch>
                                                </el-form-item>
                                            </template>
                                        </el-table-column>
                                        <el-table-column
                                            v-else-if="li.isShow === 1"
                                            :prop="li.name"
                                            :label="li.columnName"
                                            min-width="160">
                                            <template v-if="li.isRequired === 1" slot="header">
                                                <span><span class="fcErr">*&#8194;</span>{{li.columnName}}</span>
                                            </template>
                                            <template slot-scope="scope">
                                                <customize-general
                                                    v-model="scope.row.domains[index]"
                                                    :customerId="customerId"
                                                    :key="scope.row.key"
                                                    :show-label="false"
                                                    :show-message="false"
                                                    inline-message
                                                    @validateField="validateField('contactRuleTableForm', `contactColumnLists.${scope.$index}.domains.${index}.originalValue`)"
                                                    :ref="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`"
                                                    :prop="`contactColumnLists.${scope.$index}.domains.${index}.originalValue`">
                                                </customize-general>
                                            </template>
                                        </el-table-column>
                                    </a>
                                </el-table>
                            </el-form>
                        </div>
                    </div>
                </dd>
            </div>
        </dl>
        <div class="attachment" ref="anchorAttachment">
            <div class="a_bar">
                <div class="f14 fc02 ti">{{langs.attachment || '附件'}}</div>
            </div>
            <div class="a_con">
                <el-row>
                    <el-col :span="shortcutStatus ? 12 : 8" v-for="(item, index) in fileList" :key="index" style="padding: 8px;">
                        <public-file-item :fileData="item" @removeFileData="removeFileData(index)" @onPreviewFile="onPreviewFile(index)" @uploadRetry="$refs.publicUpload.handleUploadRetry(item)"></public-file-item>
                    </el-col>
                    <el-col :span="shortcutStatus ? 12 : 8" style="padding: 8px;height: 116px;">
                        <public-upload class="file-list" v-show="fileList.length < attachmentMax" ref="publicUpload"
                            :filetype="17"
                            :beyondStatus="fileList.length >= attachmentMax"
                            :maxSize="30"
                            @uploadStart="handleUploadStart"
                            @uploadProgress="handleUploadProgress"
                            @uploadSuccess="handleUploadSuccess"
                            @uploadFail="handleUploadFail">
                        </public-upload>
                    </el-col>
                </el-row>
            </div>
        </div>
    </el-scrollbar>
    <!-- 锚点导航条 -->
    <anchor :anchorList="anchorList" :anchorActiveName="anchorActiveName" @scrollToAnchor="scrollToAnchor"></anchor>
    <div class="footer_bar">
        <div class="btn_group">
            <el-button class="JOINF f12" @click="cancel">{{langs.cancel || '取消'}}</el-button>
            <!--<el-button class="JOINF f12" type="primary">{{langs.share || '共享'}}</el-button>-->
            <el-button class="JOINF f12" type="primary" @click="save(0)">{{langs.save || '保存'}}</el-button>
            <el-button v-if="!shortcutStatus" class="JOINF f12" type="primary" @click="save(1)">{{langs.saveAndNew || '保存并新建'}}</el-button>
        </div>
    </div>
    <!-- 图片预览 -->
    <lightbox :images="images" ref="lightbox"></lightbox>
</div>
</template>

<script>
import { mapGetters } from 'vuex'
import Clickoutside from 'element-ui/src/utils/clickoutside'
import customerApi from 'api/customerApi'
import businessApi from 'api/businessApi'
import customerCheck from '@/views/customer/components/api/customerCheck.js'
import customizeVerify from '@/components/customize/customizeVerify.js'
import customizeFieldProcessing from '@/components/customize/customizeFieldProcessing.js'
import getContactModels from '@/components/customize/getContactModels.js'
import CustomizeGeneral from '@/components/customize/CustomizeGeneral'
import { parameterId } from '@/common/js/config.js';

import Confirm from '@/components/ui/confirm'
import Anchor from '@/views/customer/components/customerDetail/Anchor'
import ApplicationCode from '@/components/ui/dialog/ApplicationCode'
// 附件上传、预览
import PublicFileItem from '@/components/fileUpload/PublicFileItem.vue'
import PublicUpload from '@/components/fileUpload/PublicUpload.vue'
import publicUploadFn from '@/components/fileUpload/PublicUpload.js'
import Lightbox from 'vue-image-lightbox'

import * as config from '@/common/js/config.js';

export default {
    name: 'CustomerEditAddBase',
    directives: { Clickoutside },
    mixins: [customizeVerify, customizeFieldProcessing, getContactModels, publicUploadFn, customerCheck],
    components: {
        CustomizeGeneral,
        Anchor,
        PublicFileItem,
        PublicUpload,
        Lightbox
    },
    data() {
        return {
            // 客户名称
            customerName: '',
            loading: false,
            tagId: '',
            /**
             * 验证状态
             * 用户进行操作后，状态=true
             */
            validateStatus: false,
            // 锚点内容
            anchorList: [],
            anchorActiveName: 'anchor0', // 当前锚点refName
            // 客户id
            customerId: 0,
            /**
             * 当前跳转的tab
             * 0:客户 1:公海 2:回收箱
             */
            currentTab: 0,
            // 类型 0:新建  1:编辑 2:查看
            type: 0,
            // 保存的类型  0:保存  1:保存并新建
            saveType: 0,
            // 规则refs对象名称前端
            ruleFormPrefix: 'ruleForm_',
            // 客户自定义数据
            ruleForm: [],
            // 客户自定义数据(Params传递)
            ruleFormParams: null,
            // 联系人数据
            CTRuleForm: {
                contactColumnLists: []
            },
            // 联系人数据(Params传递)
            CTRuleFormParams: null,
            // 联系人默认数据
            contactDefaultForm: {
                domains: [],
                num: 0
            },
            // 下一跟进周期
            displayLastFollowTime: null,
            fileList: [],
            attachmentMax: config.ATTACHMENT_MAX,
            /**
             * 快捷状态
             */
            shortcutStatus: false,
            // 是否修改
            isAlter: false,
            vShow: {
                companyInfo: false,
                manageInfo: false,
                extendInfo: false,
                // 锚点展开
                anchorBox: false,
                // 联系人资料列表模式 1-卡片|2-列表
                contactMode: 1,
                // 当前选中的联系人
                contactItemActive: '',
                // 联系人全选
                contactAll: false,
                // 1-时间轴模式|2-列表
                otherMode: 1,
                // 其他信息列表-当前显示标识
                otherActivated: '1',
            },
            // 保存……状态
            doSaving: false,
            // 顶部paddingTop
            barPT: 0,
            images: [], // 预览图片
        }
    },
    computed: {
        langs() {
            let $t = this.$t.bind(this);
            return Object.assign({}, $t('base'), $t('button'), $t('title'), $t('msg'), $t('ui.customize'), $t('customer.base'), $t('customer.msg'), $t('customer.customerListEditAdd'))
        },
        ...mapGetters({
            customerAddData: 'customer/getCustomerAddData',
            customerShortcutAddData: 'customer/getCustomerShortcutAddData',
            navTags: 'navTag/getNavTags',
            delCueParams: 'navTag/getAddOrEditParams',
            closeEvent: 'navTag/getAddOrEditUpdate',
            customerSelectedData: 'customer/getCustomerSelectedData'
        }),
    },
    mounted() {
        this.barPT = this.$refs.elscrollbar.wrap.getBoundingClientRect().top + 40
        // 编辑
        if (this.type === 1 || this.type === 4) {
            this.customerId = this.getNavLinkParams(this.navTags, 'id')
            this.currentTab = this.getNavLinkParams(this.navTags, 'currentTab')
            this.initAttachmentData(this.customerId) // 获取附件列表
        }
        this.ruleFormParams = this.getNavLinkParams(this.navTags, 'ruleForm')
        this.CTRuleFormParams = this.getNavLinkParams(this.navTags, 'CTRuleForm')
        this.tagId = this.getNavLinkParams(this.navTags, 'tagId')
        this.init()
        this.addNativeEvents()
    },
    methods: {
        init() {
            /**
             * 新建时，
             * 从缓存中取新建客户数据
             */
            if (this.type === 0 && this.customerAddData && this.customerAddData.length > 0) {
                this.vShow.contactMode = 1
                this.parseCustomerData(JSON.parse(JSON.stringify(this.customerAddData)))
            } else if (this.type === 3 && this.customerShortcutAddData && this.customerShortcutAddData.length > 0) {
                this.vShow.contactMode = 1
                this.parseCustomerData(JSON.parse(JSON.stringify(this.customerShortcutAddData)))
            } else {
                this.getCustomerDetail()
            }
        },
        initAttachmentData(customerId) {
            let param = {
                page: false,
                id: customerId
            }
            customerApi.getCustomerAttachments(param).then(({ data: response }) => {
                if (response.success) {
                    let res = response.data || []
                    this.fileList = this.transferFileData(res)
                }
            })
        },
        getCustomerDetail() {
            this.loading = true
            let param = {
                id: this.customerId,
                type: this.type
            }
            customerApi.getCustomerDetail(param).then(({ data: res }) => {
                if (res.success) {
                    // 新建缓存数据
                    if (this.type === 0) {
                        this.$store.commit('customer/setCustomerAddData', JSON.parse(JSON.stringify(res.data)))
                    } else if (this.type === 3) {
                        this.$store.commit('customer/setCustomerShortcutAddData', JSON.parse(JSON.stringify(res.data)))
                    }
                    this.parseCustomerData(res.data)
                } else {
                    this.$message.error(res.errMsg || '操作失败')
                }
                this.$nextTick(()=> {
                    this.loading = false
                })
            })
        },
        /**
         * 解析客户详情
         * @param {Array} CData
         */
        async parseCustomerData(CData) {
            if (CData) {
                this.anchorList = []
                this.contactDefaultForm.domains = []
                this.contactDefaultForm.num = 0
                let resultData = CData
                // 规则列表，临时数据
                let RFList = []
                resultData.forEach((list, index) => {
                    let fildArr = [];
                    if (list.categoryName === '联系人') {
                        // 生成新建联系人数据
                        let listDomains = list.columnList.sort(this.columnSort)
                        listDomains.forEach((item) => {
                            let TItem = this.parseCustomColumn(item)
                            if (TItem.isShow === 1) this.contactDefaultForm.num++
                            /**
                             * 1、状态 1:默认启用
                             * 2、状态 1:默认男
                             */
                            if (item.id === 'flag' && item.originalValue === null) {
                                item.originalValue = 1
                            } else if (item.id === 'sex' && item.originalValue === null) {
                                item.originalValue = 1
                            }
                            /**
                             * 默认联系人信息写入校验信息后。
                             * 新建时，JSON.stringify转换数据后。验证接口会无效。
                             */
                            this.contactDefaultForm.domains.push(TItem)
                        })
                        /**
                         * 1、新建；
                         * 2、联系人列表不空时；
                         * 3、编辑时，联系人列表空时；
                         */
                        if (this.type === 0) {
                            /**
                             * 快捷传递参数
                             * {expand, expand, key, defaultContact, id, {value:显示值, originalValue:id}}
                             */
                            if (this.CTRuleFormParams !== '' && this.CTRuleFormParams.length > 0) {
                                this.CTRuleFormParams.forEach((item)=> {
                                    let domains = JSON.parse(JSON.stringify(this.contactDefaultForm.domains))
                                    let num = this.contactDefaultForm.num
                                    domains.forEach((item2)=> {
                                        item2.rule = this.generateRule(item2)
                                        this.addValidateAcquaintance({
                                            callback: this.customerCheck,
                                            customerId: this.customerId,
                                            data: item2,
                                            type: 1
                                        })
                                        if (item[item2.id]) {
                                            item2.value = item[item2.id].value
                                            item2.originalValue = item[item2.id].originalValue
                                        }
                                    })
                                    this.CTRuleForm.contactColumnLists.push({
                                        domains: domains,
                                        expand: item.expand,
                                        num: num,
                                        checked: item.checked,
                                        key: item.key,
                                        defaultContact: item.defaultContact
                                    })
                                })
                            } else {
                                console.log(this.getNavLinkParams(this.navTags, 'contacts'))
                                // 初始化多个联系人
                                if (Array.isArray(this.getNavLinkParams(this.navTags, 'contacts'))) {
                                    let contacts = this.getNavLinkParams(this.navTags, 'contacts');
                                    let num = this.contactDefaultForm.num
                                    contacts.forEach(it => {
                                        let domains = JSON.parse(JSON.stringify(this.contactDefaultForm.domains));
                                        domains.forEach(item => {
                                            if (item.id === 'name') {
                                                item.originalValue = it.contactName;
                                            } else if (item.id === 'email') {
                                                item.originalValue = it.contactEmail;
                                            }
                                            this.addValidateAcquaintance({
                                                callback: this.customerCheck,
                                                customerId: this.customerId,
                                                data: item,
                                                type: 1
                                            })
                                        })
                                        this.CTRuleForm.contactColumnLists.unshift({
                                            domains: domains,
                                            expand: false,
                                            num: num,
                                            checked: false,
                                            key: new Date().getTime(),
                                            defaultContact: this.CTRuleForm.contactColumnLists.length === 0 ? 1 : 0
                                        })
                                    })
                                } else {
                                    let domains = JSON.parse(JSON.stringify(this.contactDefaultForm.domains))
                                    let num = this.contactDefaultForm.num
                                    domains.forEach((item)=> {
                                        item.rule = this.generateRule(item)
                                        /**
                                         * fromEmailtype 表示从新邮件跳转过来
                                         * new 新增客户
                                         * edit 新增联系人客户
                                         */
                                        if (item.id === 'name' && (this.getNavLinkParams(this.navTags, 'fromEmailtype') === 'new' || this.getNavLinkParams(this.navTags, 'contactEmail') !== '')) {
                                            item.originalValue = this.getNavLinkParams(this.navTags, 'contactEmail').split('@')[0]
                                        } else if (item.id === 'email' && (this.getNavLinkParams(this.navTags, 'fromEmailtype') === 'new' || this.getNavLinkParams(this.navTags, 'contactEmail') !== '')) {
                                            item.originalValue = this.getNavLinkParams(this.navTags, 'contactEmail')
                                        }
                                        this.addValidateAcquaintance({
                                            callback: this.customerCheck,
                                            customerId: this.customerId,
                                            data: item,
                                            type: 1
                                        })
                                    })
                                    this.CTRuleForm.contactColumnLists.unshift({
                                        domains: domains,
                                        expand: false,
                                        num: num,
                                        checked: false,
                                        key: new Date().getTime(),
                                        defaultContact: this.CTRuleForm.contactColumnLists.length === 0 ? 1 : 0
                                    })
                                }
                            }
                        } else if (list.columnLists && list.columnLists.length > 0) {
                            // 解析联系人数据
                            list.columnLists.forEach((item)=> {
                                // 获取联系人id
                                let contactId
                                // 默认联系人状态
                                let defaultContact = 0
                                item.some((DList)=> {
                                    if (DList.id === 'id') contactId = DList.value
                                    return DList.id === 'id'
                                })
                                let DListArr = []
                                listDomains = item.sort(this.columnSort)
                                listDomains.forEach((DList)=> {
                                    let TItem = this.parseCustomColumn(DList)
                                    /**
                                     * 写入校验客户信息
                                     */
                                    this.addValidateAcquaintance({
                                            callback: this.customerCheck,
                                            customerId: this.customerId,
                                            contactId: contactId,
                                            data: TItem,
                                            type: 1
                                        })
                                    /**
                                     * 1、默认联系人
                                     *      0:否；1:是
                                     * 2、状态
                                     *      新建时，默认启用
                                     */
                                    if (TItem.id === 'defaultContact' && TItem.originalValue) {
                                        defaultContact = TItem.originalValue
                                    }
                                    DListArr.push(TItem)
                                })
                                fildArr.push({
                                    domains: DListArr,
                                    expand: false,
                                    checked: false,
                                    num: this.contactDefaultForm.num,
                                    key: parseInt(Math.random() * 100000000),
                                    id: contactId,
                                    defaultContact: defaultContact
                                })
                            })
                            /**
                             * 快捷传递参数
                             * {expand, expand, key, defaultContact, id, {value:显示值, originalValue:id}}
                             */
                            if (this.CTRuleFormParams !== '' && this.CTRuleFormParams.length > 0) {
                                this.CTRuleFormParams.forEach((item)=> {
                                    /**
                                     * 1、已存在联系人
                                     * 2、新建联系人
                                     */
                                    if (item.id) {
                                        for (let i = 0; i < fildArr.length; i++) {
                                            if (fildArr[i].id === item.id) {
                                                fildArr[i].domains.forEach((item2)=> {
                                                    if (item[item2.id]) {
                                                        item2.value = item[item2.id].value
                                                        item2.originalValue = item[item2.id].originalValue
                                                    }
                                                })
                                                fildArr[i].expand = item.expand
                                                fildArr[i].checked = item.checked
                                                fildArr[i].num = item.num
                                                fildArr[i].key = item.key
                                                fildArr[i].defaultContact = item.defaultContact
                                                break
                                            }
                                        }
                                    } else {
                                        let domains = JSON.parse(JSON.stringify(this.contactDefaultForm.domains))
                                        domains.forEach((item2)=> {
                                            item2.rule = this.generateRule(item2)
                                            this.addValidateAcquaintance({
                                                callback: this.customerCheck,
                                                customerId: this.customerId,
                                                data: item2,
                                                type: 1
                                            })
                                            if (item[item2.id]) {
                                                item2.value = item[item2.id].value
                                                item2.originalValue = item[item2.id].originalValue
                                            }
                                        })
                                        this.CTRuleForm.contactColumnLists.push({
                                            domains: domains,
                                            expand: item.expand,
                                            checked: item.checked,
                                            key: item.key,
                                            defaultContact: item.defaultContact
                                        })
                                    }
                                })
                            }
                            this.CTRuleForm = {
                                contactColumnLists: fildArr
                            }
                        } else {
                            this.addContact()
                        }
                        RFList.push({
                            categoryName: list.categoryName,
                            refName: 'anchor' + index,
                        })
                    } else if (list.categoryName && list.columnList) {
                        // 有数据
                        if (list.columnList.length > 0) {
                            let listDomains = list.columnList.sort(this.columnSort)
                            listDomains.forEach((item) => {
                                let TItem = this.parseCustomColumn(item)
                                this.addValidateAcquaintance({
                                    callback: this.customerCheck,
                                    customerId: this.customerId,
                                    data: TItem,
                                    type: 0
                                })
                                /**
                                 * 快捷传递参数
                                 * {value:显示值, originalValue:id}
                                 */
                                if (
                                    this.ruleFormParams !== ''
                                    && this.ruleFormParams[list.categoryName]
                                    && this.ruleFormParams[list.categoryName][item.id]) {
                                    TItem.value = this.ruleFormParams[list.categoryName][item.id].value
                                    TItem.originalValue = this.ruleFormParams[list.categoryName][item.id].originalValue
                                }
                                // 添加，需要显示的字段
                                if (TItem.isShow === 1) {
                                    fildArr.push(TItem)
                                }
                                // fromEmailtype 表示从新邮件跳转过来
                                // new 新增客户
                                // edit 新增联系人客户
                                if (TItem.id === 'name' && (this.getNavLinkParams(this.navTags, 'fromEmailtype') === 'new' || this.getNavLinkParams(this.navTags, 'contactEmail') !== '')) {
                                    TItem.originalValue = this.getNavLinkParams(this.navTags, 'contactEmail')
                                }
                                if (this.getNavLinkParams(this.navTags, 'from') === 'globaleagle') {
                                    if (TItem.id === 'name' && this.getNavLinkParams(this.navTags, 'company')) {
                                        TItem.originalValue = this.getNavLinkParams(this.navTags, 'company');
                                    }
                                    if (TItem.id === 'webSite' && this.getNavLinkParams(this.navTags, 'website')) {
                                        TItem.originalValue = this.getNavLinkParams(this.navTags, 'website');
                                    }
                                }
                                if (TItem.id === 'displayLastFollowTime') {
                                    this.displayLastFollowTime = TItem
                                }
                                // 商业数据
                                if (this.getNavLinkParams(this.navTags, 'from') === 'data' && this.getNavLinkParams(this.navTags, 'baseInfo')) {
                                    let params = this.navTags.find(it => it.active).params;
                                    if (TItem.id === 'address' && params.address) {
                                        TItem.originalValue = params.address;
                                    }
                                    if (TItem.id === 'name' && params.company) {
                                        TItem.originalValue = params.company;
                                    }
                                    if (TItem.id === 'displayRegion' && params.baseInfo.countryId) {
                                        this.$refs.countrySelect.focusHandler();
                                        TItem.originalValue = params.baseInfo.countryId;
                                    }
                                    if (TItem.id === 'faceBookCmpMain' && params.baseInfo.facebookWebSite) {
                                        TItem.originalValue = params.baseInfo.facebookWebSite;
                                    }
                                    if (TItem.id === 'twitterCmpMain' && params.baseInfo.twitterWebSite) {
                                        TItem.originalValue = params.baseInfo.twitterWebSite;
                                    }
                                    if (TItem.id === 'fax' && params.fax) {
                                        TItem.originalValue = params.fax;
                                    }
                                    if (TItem.id === 'telephone' && params.telephone) {
                                        TItem.originalValue = params.telephone;
                                    }
                                    if (TItem.id === 'webSite' && params.website) {
                                        TItem.originalValue = params.website;
                                    }
                                    console.log(params);
                                }
                            })
                        }
                        RFList.push({
                            categoryName: list.categoryName,
                            refName: 'anchor' + index,
                            expand: true,
                            domains: fildArr
                        })
                    }
                })
                this.ruleForm = RFList
                // 从邮件跳转 编辑客户-新增联系人
                if (this.getNavLinkParams(this.navTags, 'fromEmailtype') === 'edit') {
                    this.addContact()
                    this.CTRuleForm.contactColumnLists.forEach((item, index)=>{
                        if (index === 0) {
                            item.domains.forEach(list=>{
                                if (list.id === 'name') {
                                    list.originalValue = this.getNavLinkParams(this.navTags, 'contactEmail').split('@')[0]
                                } else if (list.id === 'email') {
                                    list.originalValue = this.getNavLinkParams(this.navTags, 'contactEmail')
                                }
                            })
                        }
                    })
                }
                // 自动客户代码
                let from = this.getParamValue('from');
                if (from === 'edm') {
                    businessApi.getCodesBusiness(parameterId.CUSTOMER).then(({ data: res }) => {
                        if (res.success) {
                            if (!res.data || res.data.length === 0) {
                                let domain = this.ruleForm.find(it => it.domains && it.domains.find(i => i.id === 'code'));
                                if (domain) {
                                    let item = domain.domains.find(it => it.id === 'code');
                                    let index = domain.domains.indexOf(item);
                                    console.error(item);
                                    this.generatedCode(item, [], this.ruleFormPrefix + domain.refName, `domains.${index}.originalValue`);
                                }
                            }
                        }
                    })
                }
                this.setAnchorList()
                // console.error('RFList', RFList)
                // console.error('CTRuleForm', this.CTRuleForm)
            }
            /**
             * 初始化修改状态
             */
            this.$nextTick(()=> {
                this.isAlter = false
            })
        },
        /**
         * 设置锚列表
         */
        setAnchorList() {
            let AList = this.ruleForm.map((item, index)=> {
                return {
                    name: item.categoryName,
                    refName: 'anchor' + index,
                    level: item.categoryName === '联系人' ? 1 : 2
                }
            })
            AList.push({
                    name: '附件',
                    refName: 'anchorAttachment',
                    level: 1
                })
            this.anchorList = AList
        },
        /**
         * 获取自定义编码数据
         * @param {Object} item 代码数据项
         */
        getGeneratedCode(item, refName, prop) {
            businessApi.getCodesBusiness(parameterId.CUSTOMER).then(({ data: res }) => {
                if (res.success) {
                    // 有申请字典
                    if (res.data.length > 0) {
                        // 弹出自定义编码弹出框
                        this.helper.modal.open(ApplicationCode, {
                            parent: this,
                            data: {
                                self: this
                            },
                            propsData: {
                                codesData: res.data
                            }
                        }).then((result) => {
                            this.generatedCode(item, result, refName, prop);
                        }, (err) => {
                            this.$message.error(err)
                        })
                    } else {
                        this.generatedCode(item, [], refName, prop);
                    }
                } else {
                    this.$message.warning(res.errMsg)
                }
            })
        },
        /**
         * 生成编码代码
         * @param {Object} item 代码数据项
         * @param {Arry} list 字典列表
         */
        generatedCode(item, list, refName, prop) {
            let param = {
                list: list,
                parameterId: parameterId.CUSTOMER
            }
            businessApi.postCodes(param).then(({ data: res }) => {
                // console.log(res)
                if (res.success) {
                    item.originalValue = res.data
                    this.validateField(refName, prop)
                } else {
                    this.$message.warning(res.errMsg)
                }
            })
        },
        /**
         * 选择联系人(卡片)
         */
        selectContact(item) {
            if (item.checked) {
                item.checked = false
                // 取消全选
                this.vShow.contactAll = false
            } else {
                item.checked = true
                /**
                 * 设置全选状态
                 * 如果有一个未全选，就不是全选状态
                 */
                this.vShow.contactAll = !this.CTRuleForm.contactColumnLists.some((list)=> {
                    return list.checked === false
                })
            }
        },
        /**
         * 联系人全选（卡片）
         */
        selectAllContact() {
            this.CTRuleForm.contactColumnLists.forEach((list)=> {
                list.checked = this.vShow.contactAll
            })
        },
        /**
         * 添加联系人
         */
        addContact() {
            let domains = JSON.parse(JSON.stringify(this.contactDefaultForm.domains))
            let num = this.contactDefaultForm.num
            domains.forEach((item)=> {
                item.rule = this.generateRule(item)
                this.addValidateAcquaintance({
                    callback: this.customerCheck,
                    customerId: this.customerId,
                    data: item,
                    type: 1
                })
            })
            this.CTRuleForm.contactColumnLists.unshift({
                domains: domains,
                expand: false,
                num: num,
                checked: false,
                key: new Date().getTime(),
                defaultContact: this.CTRuleForm.contactColumnLists.length === 0 ? 1 : 0
            })
        },
        /**
         * 设为默认联系人
         */
        setDefaultContact(index, switchBtn) {
            /**
             * 1、切换按钮
             * 2、取消默认
             * 3、设为默认
             */
            if (switchBtn) {
                this.CTRuleForm.contactColumnLists.forEach((item, pos)=> {
                    if (pos !== index) {
                        item.defaultContact = 0
                    }
                })
            } else if (this.CTRuleForm.contactColumnLists[index].defaultContact === 1) {
                this.CTRuleForm.contactColumnLists[index].defaultContact = 0
            } else {
                this.CTRuleForm.contactColumnLists.forEach((item)=> {
                    item.defaultContact = 0
                })
                this.CTRuleForm.contactColumnLists[index].defaultContact = 1
            }
        },
        /**
         * 删除联系人
         */
        delContact(force) {
            // 删除数量
            let delNumber = 0
            // 删除确认
            let delConfirm = false
            if (this.vShow.contactMode === 1) {
                // 卡片模式
                for (let i = this.CTRuleForm.contactColumnLists.length - 1; i >= 0; i--) {
                    if (this.CTRuleForm.contactColumnLists[i].checked) {
                        delNumber++
                        if (this.CTRuleForm.contactColumnLists[i].id && !force) {
                            delConfirm = true
                        } else {
                            this.CTRuleForm.contactColumnLists.splice(i, 1)
                        }
                    }
                }
            } else {
                // 列表模式
                let contactObj = {}
                this.$refs.contactTable[0].selection.forEach((item)=> {
                    contactObj[item.key] = true
                })
                for (let i = this.CTRuleForm.contactColumnLists.length - 1; i >= 0; i--) {
                    if (contactObj[this.CTRuleForm.contactColumnLists[i].key]) {
                        delNumber++
                        if (this.CTRuleForm.contactColumnLists[i].id && !force) {
                            delConfirm = true
                        } else {
                            this.CTRuleForm.contactColumnLists.splice(i, 1)
                        }
                    }
                }
            }
            if (delNumber === 0) this.$message.warning(this.langs.pleaseSelectContact)
            // 清空后，新建一条空数据
            if (this.CTRuleForm.contactColumnLists.length === 0) {
                this.addContact()
            }
            // 删除确认
            if (delConfirm && !force) {
                this.helper.modal.open(Confirm, {
                    data: {
                        self: this
                    },
                    propsData: {
                        label: this.langs.confirmDeletionContact,
                        msg: this.langs.confirmDeletionContactCue,
                        confirmButtonText2: this.langs.yes,
                        cancelButtonText: this.langs.cancel,
                        type: 'info',
                        width: 500
                    }
                }).then(()=>{
                    this.delContact(true)
                }, () => {
                })
            }
        },
        /**
         * 展开联系人
         */
        expandConstact(pos) {
            this.CTRuleForm.contactColumnLists.forEach((item, index)=>{
                if (index === pos) {
                    item.expand = true
                } else {
                    item.expand = false
                }
            })
        },
        /**
         * 收起联系人
         */
        collapseConstact(item) {
            item.expand = false
        },
        handleClickOutside1() {
            // 锚点展开
            this.vShow.anchorBox = false;
        },
        handleClickOutside2() {
            this.vShow.cIndexCurrent = false;
        },
        /**
         * 客户类型
         */
        changeDisplayType(type) {
            if (!this.displayLastFollowTime) return false
            let params = {
                id: this.customerId,
                type: type
            }
            customerApi.getCustomerGetLastFollowTime(params).then(({ data: res })=>{
                if (res.success) {
                    this.displayLastFollowTime.originalValue = Number(res.data)
                }
            })
        },
        /**
         * 是否多行
         */
        isMultiline(row) {
            let MStatus = false
            if (row.type === 10 && row.property === 2) {
                MStatus = true
            } else if (row.type === 12 && row.property === 2) {
                MStatus = true
            }
            return MStatus
        },
        /**
         * 添加原生事件
         */
        addNativeEvents() {
            window.addEventListener('resize', this.resize, false)
            this.$refs.elscrollbar.wrap.addEventListener('scroll', this.scrollFn, false);
        },
        resize() {},
        /**
         * 滚动到锚点位置
         * @param  {string} refName 当前锚点名称
         * @param  {number} idx     当前锚点下标
         */
        scrollToAnchor(refName, idx) {
            this.anchorActiveName = refName
            this.anchorSetting = true
            let node = this.$refs[this.anchorList[idx].refName]
            if (node.length) node = node[0]
            node.scrollIntoView({ behavior: 'smooth' })
            setTimeout(() => {
                this.anchorSetting = false
            }, 500)
        },
        /**
         * 获取contacts模块数据
         * @description 保存操作
         */
        getContactParam() {
            let contactD = []
            this.CTRuleForm.contactColumnLists.forEach((list)=> {
                contactD.push(this.getContactModels(list))
            })
            // console.log('GetContact: ', contactD)
            return contactD;
        },
        /**
         * 获取contacts模块数据（传参）
         */
        getContactModelsData(data) {
            let dataObj = []
            data.contactColumnLists.forEach((item, index)=> {
                dataObj[index] = {}
                dataObj[index].checked = item.checked
                dataObj[index].defaultContact = item.defaultContact
                dataObj[index].expand = item.expand
                dataObj[index].key = item.key
                if (item.id) dataObj[index].id = item.id
                item.domains.forEach((domains)=> {
                    dataObj[index][domains.id] = {
                        value: domains.value,
                        originalValue: domains.originalValue,
                    }
                })
            })
            return dataObj
        },
        /**
         * 获取models模块数据
         * @description 保存操作
         */
        getModels() {
            let models = []
            // 解析数据
            this.ruleForm.forEach((list)=> {
                if (list.domains) {
                    list.domains.forEach((item)=> {
                        let obj = this.parseModelsData(item)
                        /**
                         * 国家选择已改变时，obj.originalValue === ''
                         */
                        if (item.id === 'displayRegion' && this.customerSelectedData.country && obj.displayValue === '') {
                            for (let i = 0; i < this.customerSelectedData.country.length; i++) {
                                if (this.customerSelectedData.country[i].id === obj.value) {
                                    obj.displayValue = this.customerSelectedData.country[i].cname + '--' + this.customerSelectedData.country[i].ename
                                }
                            }
                        } else if (item.id === 'name') {
                            this.customerName = item.originalValue
                        }
                        /**
                         * 不可编辑的不提交
                         * code:客户编码必须提交
                         */
                        if (item.force || item.isReadonly === 0) {
                            models.push(obj)
                        } else if (item.id === 'code') {
                            models.push(obj)
                        }
                    })
                }
            })
            // console.log('GetModels: ', models)
            return models;
        },
        /**
         * 获取客户数据（传参）
         * @param data
         * @returns 生成客户自定义字段数据
         */
        getModelsData(data) {
            let dataObj = {}
            data.forEach((item)=> {
                if (item.categoryName !== '联系人' && item.domains.length > 0) {
                    dataObj[item.categoryName] = {}
                    item.domains.forEach((domains)=> {
                        dataObj[item.categoryName][domains.id] = {
                            value: domains.value,
                            originalValue: domains.originalValue,
                        }
                    })
                }
            })
            return dataObj
        },
        /**
         * 获取附件数据
         * @description 保存操作
         */
        getAttachments() {
            return this.fileList.map(item => {
                return {
                    fileCode: item.code,
                    fileName: item.name,
                    fileSize: item.size,
                    fileSuffix: item.type,
                    id: item.id
                }
            })
        },
        /**
         * 保存前
         */
        saveBeforeFun() {},
        saveFun() {},
        /**
         * 保存/保存并新建
         * @param {Number} type 0:保存 1:保存并新建
         */
        save(type) {
            if (this.doSaving) return false
            this.doSaving = true
            this.saveType = type
            let validate = true
            this.ruleForm.forEach((list, index)=> {
                let SV = true
                /**
                 * 1、联系人
                 * 2、其他类型，显示项 > 0
                 * 3、其他默认验证通过
                 */
                if (list.categoryName === '联系人') {
                    if (this.vShow.contactMode === 1) {
                        /**
                         * 卡片模式
                         * 遍历卡片验证
                         */
                        this.$refs.contactRuleForm.forEach((CRFList) => {
                            CRFList.validate((valid, obj) => {
                                if (!valid) {
                                    let objKeys = Object.keys(obj)
                                    this.expandConstact(Number(objKeys[0].split('.')[1]))
                                    if (validate) this.gotoValidate(obj)
                                }
                                if (SV) SV = valid
                            })
                        })
                    } else {
                        // 列表模式
                        this.$refs.contactRuleTableForm[0].validate((valid, obj) => {
                            if (!valid) {
                                if (validate) this.gotoValidate(obj)
                            }
                            SV = valid
                        })
                    }
                } else if (list.domains.length > 0) {
                    this.$refs[this.ruleFormPrefix + list.refName][0].validate((valid, obj) => {
                        if (!valid) {
                            if (index === 0 && validate) {
                                this.gotoValidate(obj, list.refName)
                            } else {
                                list.expand = true
                                if (validate) {
                                    this.$nextTick(()=> {
                                        this.gotoValidate(obj, list.refName)
                                    })
                                }
                            }
                        }
                        SV = valid
                    })
                } else {
                    SV = true
                }
                if (validate) validate = SV
            })
            // console.log('validate', validate)
            if (validate) {
                // console.log('通过验证')
                // this.getModels()
                // this.getContactParam()
                this.saveFun()
            } else {
                this.doSaving = false
            }
        },
        /**
         * 跳转到未通过验证字段
         * @param obj {Object} 验证没有通过字段
         * @param refName {String} refs名称前缀
         */
        gotoValidate(obj, refName) {
            let objKeys = Object.keys(obj)
            if (refName) {
                this.$refs.elscrollbar.wrap.scrollTop = this.$refs.elscrollbar.wrap.scrollTop + this.$refs[refName + objKeys[0]][0].$el.getBoundingClientRect().top - this.barPT
            } else {
                this.$refs.elscrollbar.wrap.scrollTop = this.$refs.elscrollbar.wrap.scrollTop + this.$refs[objKeys[0]][0].$el.getBoundingClientRect().top - this.barPT
            }
        },
        /**
         * 全屏打开
         */
        fullscreenOpen() {},
        /**
         * 取消
         */
        cancel() {
            this.closeActuve()
        },
        /**
         * 关闭二次确认
         */
        closeConfirm() {
            // 进行编辑操作
            if (!this.isAlter) {
                this.close()
                return false
            }
            this.helper.modal.open(Confirm, {
                data: {
                    self: this
                },
                propsData: {
                    label: this.langs.cue,
                    msg: this.langs.draftSaveCloseCue,
                    confirmButtonText2: this.langs.yes,
                    confirmButtonText: this.langs.no,
                    cancelButtonText: this.langs.cancel,
                    type: 'info',
                    width: 500
                }
            }).then((result)=>{
                if (result === 'sure2') {
                    this.save(0)
                } else if (result === 'sure') {
                    this.close()
                }
            }, () => {
            })
        },
        close() {
            if (this.getNavLinkParams(this.navTags, 'tagId') === this.tagId) {
                this.closeActuve()
            } else {
                this.$store.dispatch('navTag/delNavTag', {
                    tagId: this.tagId
                })
            }
        },
        // 关闭当前窗口
        closeActuve() {
            this.$store.dispatch('navTag/delNavActiveTag')
        },
        /**
         * 滚动时更新锚点位置
         */
        scrollFn() {
            if (!this.anchorSetting) {
                let pos = this.$refs.elscrollbar.wrap.scrollTop
                let anchorList = this.anchorList
                let name = anchorList[anchorList.length - 1].refName
                for (let i = anchorList.length - 1; i >= 0; i--) {
                    let refName = anchorList[i].refName
                    let node = this.$refs[refName]
                    if (node.length) node = node[0]
                    if (node && node.offsetTop + node.clientHeight <= pos) {
                        break
                    }
                    name = refName
                }
                this.anchorActiveName = name
            }
        },
    },
    watch: {
        /**
         * 关闭事件
         */
        closeEvent() {
            if (this.delCueParams.tagId === this.tagId) {
                this.closeConfirm()
            }
        },
        ruleForm: {
            handler() {
                this.isAlter = true
            },
            deep: true
        },
        CTRuleForm: {
            handler() {
                this.isAlter = true
            },
            deep: true
        },
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.resize, false);
        this.$refs.elscrollbar.wrap.removeEventListener('scroll', this.scrollFn, false);
    },
}
</script>

<style lang="scss" scoped>
.customer_edit_add_vue {
    position: relative;
    border: 1px solid $table-border-color;
    border-top: none;
    height: 100%;
    padding-bottom: 60px;
    &.shortcut{
        padding-top: 52px;
        border: none;
        .a_s_g_p_bar{
            display: block;
        }
        .c-content{
            dt {
                &.has-icon-right{
                    padding-right: 22px;
                }
            }
            dd {
                padding: 36px 61px 23px 0;
                .base-info{
                    .demo-ruleForm{
                        &>ul {
                            li {
                                width: 100%;
                                padding-left: 136px;
                                padding-right: 0;
                            }
                        }
                    }
                }
                &.contact-info-content{
                    .tool_bar{
                        padding-left: 24px;
                        .b_btn_group{
                            right: 24px;
                        }
                    }
                    .contact-cards{
                        padding: 0 22px;
                        .card-item{
                            width: 100%;
                            &:nth-of-type(2n + 1){
                                padding-left: 0;
                                padding-right: 0;
                            }
                            &:nth-of-type(2n + 2){
                                padding-left: 0;
                                padding-right: 0;
                            }
                        }
                    }
                }
            }
        }
        .attachment {
            .a_con {
                padding: 14px;
            }
        }
    }
    .a_s_g_p_bar{
        position: absolute;
        display: none;
        width: 100%;
        height: 52px;
        line-height: 52px;
        top: 0;
        left: 0;
        padding-left: 22px;
        border-bottom: 1px solid $border-color-base;
        .btn{
            position: absolute;
            top: 0;
            right: 14px;
            .btn_box{
                padding: 6px;
            }
        }
    }
    .c-content {
        .w_100 {
            width: 100%;
        }
        dt {
            position: relative;
            background-color: $background-color-base5;
            height: 42px;
            line-height: 42px;
            padding-left: 22px;
            margin-bottom: 2px;
            &.has-icon-right{
                padding-right: 14px;
            }
            .arrow {
                position: absolute;
                right: 20px;
                transition: transform 0.3s;
                -moz-transition: transform 0.3s;
                -webkit-transition: transform 0.3s;
                -o-transition: transform 0.3s;
                &.develop{
                    -webkit-transform: rotate(180deg);
                    -moz-transform: rotate(180deg);
                    -ms-transform: rotate(180deg);
                    -o-transform: rotate(180deg);
                    transform: rotate(180deg);
                }
            }
        }

        dd {
            padding: 36px 9.9% 23px 9.9%;
            background-color: $background-color-base;
            .base-info{
                .demo-ruleForm{
                    width: 100%;
                    &>ul {
                        li {
                            position: relative;
                            float: left;
                            width: 50%;
                            height: 26px;
                            list-style: none;
                            padding-left: 13.74%;
                            padding-right: 6.23%;
                            white-space: nowrap;
                            overflow: hidden;
                            -ms-text-overflow: ellipsis;
                            text-overflow: ellipsis;

                            &.item1 {
                                position: relative;
                                height: 50px;

                                .unit {
                                    position: absolute;
                                    display: inline-block;
                                    line-height: 32px;
                                    /*width: 10.5%;*/
                                    right: 10px;
                                    top: 2px;
                                }

                                .icon_code {
                                    position: absolute;
                                    width: 16px;
                                    height: 16px;
                                    line-height: 100%;
                                    right: 0;
                                    top: 8px;
                                    margin-right: 9px;
                                    cursor: pointer;
                                }

                                .lost_rrder {
                                    width: 96px;
                                }
                            }

                            &.item2 {
                                min-height: 50px;
                                height: auto;
                                position: relative;
                            }

                            &.item3 {
                                line-height: 32px;
                            }
                        }
                    }
                }
            }
            &.contact-info-content{
                padding: 0;
                .tool_bar{
                    position: relative;
                    height: 70px;
                    padding: 32px 30px 20px 30px;
                    .b_btn_group{
                        position: absolute;
                        right: 30px;
                        top: 20px;
                        .t_btn{
                            width: 32px;
                            padding: 0;
                        }
                    }
                }
                .contact-cards{
                    display: table;
                    width: 100%;
                    padding: 0 30px;
                    .card-item{
                        width: 50%;
                        float: left;
                        position: relative;
                        padding: 0 10px 20px 10px;
                        &:nth-of-type(2n + 1){
                            padding-left: 0;
                        }
                        &:nth-of-type(2n + 2){
                            padding-right: 0;
                        }
                        &.hasExpand{
                            ul{
                                li{
                                    &.display{
                                        display: block;
                                    }
                                }
                            }
                            .d_btn{
                                display: inline;
                            }
                        }
                        &.expand.hasExpand{
                            z-index: 2;
                            ul{
                                height: auto;
                            }
                        }
                        ul{
                            position: relative;
                            padding-top: 20px;
                            border-radius: 8px;
                            background-color: $background-color-base;
                            border: 1px solid $border-color-base-regular3;
                            padding-right: 45px;
                            overflow: hidden;
                            &:hover, &.checked{
                                -webkit-box-shadow: 0px 0px 9px 0px rgba(23, 131, 251, 0.44);
                                -moz-box-shadow: 0px 0px 9px 0px rgba(23, 131, 251, 0.44);
                                box-shadow: 0px 0px 9px 0px rgba(23, 131, 251, 0.44);
                            }
                            li{
                                position: relative;
                                float: left;
                                width: 50%;
                                height: 26px;
                                list-style: none;
                                padding-left: 18%;
                                // padding-right: 16px;
                                white-space: nowrap;
                                overflow: hidden;
                                -ms-text-overflow: ellipsis;
                                text-overflow: ellipsis;
                                /deep/ .el-form-item {
                                    label.el-form-item__label {
                                        width: 36%;
                                        padding-right: 10px;
                                        padding-left: 10px;
                                        text-overflow: ellipsis;
                                        overflow: hidden;
                                    }
                                }
                                &.item1 {
                                    position: relative;
                                    height: 48px;
                                    .unit {
                                        position: absolute;
                                        display: inline-block;
                                        line-height: 32px;
                                        /*width: 10.5%;*/
                                        right: 10px;
                                        top: 2px;
                                    }
                                    .icon_code {
                                        position: absolute;
                                        width: 16px;
                                        height: 16px;
                                        line-height: 100%;
                                        right: 0;
                                        top: 8px;
                                        margin-right: 9px;
                                        cursor: pointer;
                                    }
                                    .lost_rrder {
                                        width: 96px;
                                    }
                                }
                                &.item2 {
                                    min-height: 38px;
                                    height: auto;
                                    position: relative;
                                }
                                &.item3 {
                                    line-height: 32px;
                                }
                                &.display {
                                    display: none;
                                    width: 100%;
                                    height: 44px;
                                    line-height: 36px;
                                    padding-left: 45px;
                                    text-align: center;
                                    .d_btn{
                                        padding:6px 4px;
                                        .icon{
                                            margin-right: 6px;
                                        }
                                        .develop {
                                            -webkit-transform: rotate(180deg);
                                            -moz-transform: rotate(180deg);
                                            -ms-transform: rotate(180deg);
                                            -o-transform: rotate(180deg);
                                            transform: rotate(180deg);
                                        }
                                    }
                                }
                            }
                            .default_btn {
                                position: absolute;
                                width: 100px;
                                height: 33px;
                                line-height: 0;
                                top: 0;
                                right: -51px;
                                text-align: center;
                                transform-origin: center top;
                                -ms-transform-origin: center top;
                                -moz-transform-origin: center top;
                                -webkit-transform-origin: center top;
                                -o-transform-origin: center top;
                                transform:rotate(45deg);
                                -ms-transform:rotate(45deg);
                                -moz-transform:rotate(45deg);
                                -webkit-transform:rotate(45deg);
                                -o-transform:rotate(45deg);
                                opacity: 0;
                                .bg{
                                    width: 100%;
                                    height: 100%;
                                    background-color: $background-color-base2;
                                }
                                .text{
                                    position: absolute;
                                    top: 0;
                                    left: 0;
                                    width: 100%;
                                    height: 100%;
                                    line-height: 42px;
                                }
                            }
                            &:hover{
                                .default_btn{
                                    opacity: 1;
                                }
                            }
                            &.default {
                                .default_btn{
                                    opacity: 1;
                                    .bg{
                                        background-color: $background-color-light;
                                        opacity: 1;
                                    }
                                }
                            }
                        }
                    }
                }
                .contact-list{
                    margin-top: 20px;
                }
                .t-pagination{
                    margin-top: 20px;
                }
            }
        }
        .t-switch-btn{
            span{
                width: 24px;
            }
        }
    }
    .attachment {
        height: 300px;
        padding-top: 21px;
        .a_bar {
            position: relative;
            height: 42px;
            line-height: 42px;
            background-color: $background-color-base5;
            padding-left: 110px;
            border-top: 1px solid $border-color-base;
            .ti {
                position: absolute;
                height: 42px;
                line-height: 42px;
                top: 0;
                left: 22px;
            }
        }
        .a_con {
            width: 100%;
            padding: 20px;
            background-color: $background-color-base;
        }
    }
    .footer_bar {
        position: absolute;
        width: 100%;
        height: 58px;
        bottom: 0;
        left: 0;
        background-color: $background-color-base;
        border-top: 1px solid $border-color-base;
        z-index: 10;
        .btn_group {
            position: absolute;
            height: 32px;
            right: 20px;
            top: 13px;
        }
    }
    /deep/ .el-button {
        padding: 0 8px;
    }
    /deep/ .el-checkbox__label {
        font-size: 12px;
    }
    .contact_table{
        /deep/ .el-input {
            .el-input__inner{
                border-color: transparent;
            }
        }
        /deep/ .el-textarea{
            .el-textarea__inner{
                border-style: none;
            }
        }
    }
    /deep/ .el-input {
        &.pointer{
            .el-input__inner{
                cursor: pointer;
            }
        }
        .el-textarea__inner {
            padding: 5px 10px;
        }
    }
    /deep/ .el-input__prefix {
        left: auto;
        right: 5px;
        .el-input__icon {
            line-height: 32px;
        }
    }
    /deep/ .el-radio {
        margin-right: 12px;
    }
    /deep/ .el-form-item {
        label.el-form-item__label {
            position: absolute;
            width: 27.48%;
            height: 32px;
            line-height: 32px;
            padding-right: 19px;
            font-size: 12px;
            color: $color-text-regular;
            top: 0;
            left: 0;
            text-align: right;
        }
        .el-form-item__content {
            line-height: 32px;
            position: relative;
            font-size: 14px;
        }
        .el-form-item__error {
            color: $error-color;
            padding-top: 2px;
        }
    }
}
</style>
