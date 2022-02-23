<template>
    <view class="u-select">
        <u-popup :maskCloseAble="maskCloseAble" mode="bottom" :popup="false" v-model="value" length="auto" :safeAreaInsetBottom="safeAreaInsetBottom" @close="close" :z-index="uZIndex">
            <view class="u-select" v-bind:style="{paddingBottom:keyBoardHieght}">
                <view class="u-select__header" @touchmove.stop.prevent="">
                    <view
                        class="u-select__header__cancel u-select__header__btn"
                        :style="{ color: cancelColor }"
                        hover-class="u-hover-class"
                        :hover-stay-time="150"
                        @tap="getResult('cancel')"
                    >
                        {{cancelText}}
                    </view>
                    <view class="u-select__header__title">
                        {{title}}
                        <!-- 自定搜索 -->
                        <!-- 未处理原标题和搜索共存情况，请勿同时使用 -->
                        <view v-if="allowSearch" class="select-search-cont">
                            <u-icon
                                name="search"
                                color="#8F9399"
                                size="28"
                            ></u-icon>
                            <input
                                class="u-input__input"
                                :type="'text'"
                                :value="searchValue"
                                :placeholder="'请输入关键字'"
                                @input="searchOptions"
                                @tap.stop="onSubscribe"
                                @blur="offSubscribe"
                            />
                        </view>
                    </view>
                    <view
                        class="u-select__header__confirm u-select__header__btn"
                        :style="{ color: moving ? cancelColor : confirmColor }"
                        hover-class="u-hover-class"
                        :hover-stay-time="150"
                        @touchmove.stop=""
                        @tap.stop="getResult('confirm')"
                    >
                        {{confirmText}}
                    </view>
                </view>
                <!-- show selected 展示已选择的值-->
                <view class="cascader-selected-value">
                    <scroll-view scroll-x="true" class="cascader-selected-value-scoll-view" @scroll="selectedScroll" :scroll-left="scrollLeft">
                        <view class="cascader-selected-value-item" :class="index == selectColumnIndex ?'cascader-selected-value-item-active':''" v-for="(item,index) in selectValue" :key="index">
                            <view class="u-line-1" @click="selectColumn(index)">{{ item.label }}</view>
                        </view>
                    </scroll-view>
                </view>
                <!-- select body -->
                <view class="u-select__body" :style="{height:selectCont_Height}">
                    <!-- @change="columnChange" -->
                    <picker-view class="u-select__body__picker-view" :value="columPicked" @pickstart="pickstart" @pickend="pickend"
                                 :indicator-style="theme.includes('easy') ? 'height:54px' : 'height:34px'">
                        <!-- easy mode 模式 picker-view 98 font 48 -->
                        <!-- 可以换成父组件控制的形式，便于统一维护方式 ,如果easymode模式放大倍数变化，此处需要重新计算-->
                        <!-- <view> -->
                        <picker-view-column :style="{'display':index == selectColumnIndex?'block':'none'}" v-for="(item, index) in columnData" :key="index">
                            <!-- <view v-if="index == selectColumnIndex"> -->
                            <view  class="u-select__body__picker-view__item" v-for="(item1, index1) in item" :key="index1" :style="{'font-size':theme.includes('easy') ? '48rpx':'32rpx'}"
                            >
                                <!-- v-bind:style="{display:searchValue ? (searchList.includes(item1[labelName]) ? '' : 'none'):''}" -->
                                <view class="u-line-1" :class="selectValue[selectColumnIndex].label == item1[labelName] ?'selected-item':''" @click="selectedItem(index1)">{{ item1[labelName] }}
                                    <u-icon v-if="selectValue[selectColumnIndex].label == item1[labelName]" name="checkbox-mark" color="#2979FF" size="28"></u-icon>
                                </view>
                            </view>
                            <!-- </view> -->
                        </picker-view-column>
                        <!-- </view> -->
                    </picker-view>
                </view>
            </view>
        </u-popup>
    </view>
</template>

<script>
	/**
     * 修改版本：本组件从uView-ui uselect 修改，仅适用于多列联动（级联字段使用）mode "mutil-column-auto" 固定，使用其他无效
	 * @tutorial http://uviewui.com/components/select.html
	 * @property {String} mode mutil-column-auto
	 * @property {Array} list 列数据，数组形式，见官网说明
	 * @property {Boolean} v-model 布尔值变量，用于控制选择器的弹出与收起
	 * @property {Boolean} safe-area-inset-bottom 是否开启底部安全区适配(默认false)
	 * @property {String} cancel-color 取消按钮的颜色（默认#606266）
	 * @property {String} confirm-color 确认按钮的颜色(默认#2979ff)
	 * @property {String} confirm-text 确认按钮的文字
	 * @property {String} cancel-text 取消按钮的文字
	 * @property {String} default-value 提供的默认选中的下标，见官网说明
	 * @property {Boolean} mask-close-able 是否允许通过点击遮罩关闭Picker(默认true)
	 * @property {String Number} z-index 弹出时的z-index值(默认10075)
	 * @property {String} value-name 自定义list数据的value属性名 1.3.6
	 * @property {String} label-name 自定义list数据的label属性名 1.3.6
	 * @property {String} child-name 自定义list数据的children属性名，只对多列联动模式有效 1.3.7
	 * @event {Function} confirm 点击确定按钮，返回当前选择的值
	 * @example <u-select v-model="show" :list="list"></u-select>
	 */

    export default {
        name: 'cascader-select',
        props: {
            // 列数据
            list: {
                type: Array,
                default() {
                    return [];
                }
            },
            // 是否显示边框
            border: {
                type: Boolean,
                default: true
            },
            // 通过双向绑定控制组件的弹出与收起
            value: {
                type: Boolean,
                default: false
            },
            // "取消"按钮的颜色
            cancelColor: {
                type: String,
                default: '#606266'
            },
            // "确定"按钮的颜色
            confirmColor: {
                type: String,
                default: '#2979ff'
            },
            // 弹出的z-index值
            zIndex: {
                type: [String, Number],
                default: 0
            },
            safeAreaInsetBottom: {
                type: Boolean,
                default: false
            },
            // 是否允许通过点击遮罩关闭Picker
            maskCloseAble: {
                type: Boolean,
                default: true
            },
            // 提供的默认选中的下标
            defaultValue: {
                type: Array,
                default() {
                    return [0];
                }
            },
            // 模式选择，mutil-column-auto-多列联动 不支持其他值
            mode: {
                type: String,
                default: 'mutil-column-auto'
            },
            // 自定义value属性名
            valueName: {
                type: String,
                default: 'value'
            },
            // 自定义label属性名
            labelName: {
                type: String,
                default: 'label'
            },
            // 自定义多列联动模式的children属性名
            childName: {
                type: String,
                default: 'children'
            },
            // 顶部标题
            title: {
                type: String,
                default: ''
            },
            // 取消按钮的文字
            cancelText: {
                type: String,
                default: '取消'
            },
            // 确认按钮的文字
            confirmText: {
                type: String,
                default: '确认'
            },
            allowSearch: { // 允许搜索
                type:Boolean,
                default:false
            },
            serveSearch:{ // 服务端搜索
                type:Object,
                default:()=>{
                    return {
                        f:0,
                        params:{}
                    };
                }
            },
            listLength:{ // 列表长度，用于直接过去columnNum
                type:Number,
                default:1
            },
            theme: {
                type: String,
                default: 'light-normal'
            },
            defaultSelectValue:{
                type: Array,
                default:()=>{
                    return [{value:'',label:'请选择'}];
                }
            }
        },
        data() {
            return {
                // 用于列改变时，保存当前的索引，下一次变化时比较得出是哪一列发生了变化
                defaultSelector: [0],
                // picker-view的数据
                columnData: [],
                // 每次队列发生变化时，保存选择的结果
                selectValue: [{value:'',label:'请选择'}],
                // 上一次列变化时的index
                lastSelectIndex: [],
                // 列数
                columnNum: 0,
                // 列是否还在滑动中，微信小程序如果在滑动中就点确定，结果可能不准确
                moving: false,
                searchValue:'', // 搜索数据
                searchList:[] ,// 搜索的结果列表
                timer:'',  // 搜索防抖计时器
                keyBoardHieght:'',
                keybordSubs:'',
                selectCont_Height:'920rpx', // 选择区域高度，默认920rpx
                selectColumnIndex:0,
                columnList:[],
                columPicked:[0],
                scrollLeft:0 // scroll view 滚动进度
            };
        },
        watch: {
            // 在select弹起的时候，重新初始化所有数据
            value: {
                immediate: true,
                handler(val) {
                    if(val) setTimeout(() => this.init(), 10);
                }
            },
        },
        computed: {
            uZIndex() {
                // 如果用户有传递z-index值，优先使用
                return this.zIndex ? this.zIndex : this.$u.zIndex.popup;
            },
        },
        methods: {
            /**
             * 选择的列数据
             */
            selectedItem(i){
                // 更新列数据
                this.columnList[this.selectColumnIndex] = i;
                this.columnChange(this.$u.deepClone(this.columnList));
            },
            selectedScroll(e){
                // console.log(e,this.scrollLeft);
            },
            /**
             * 进行列选择
             */
            selectColumn(i){
                this.selectColumnIndex = i;
                // 处理scroll-view可视区域
                let scrollViewWidth;
                wx.createSelectorQuery().in(this).selectAll('.cascader-selected-value-scoll-view').boundingClientRect().exec(res=>{
                    scrollViewWidth = res[0][0].width;
                });
                wx.createSelectorQuery().in(this).selectAll('.cascader-selected-value-item').boundingClientRect().exec(res=>{
                    const selected = res[0][i]; // 选中元素
                    const elVisable = selected.left + selected.width; // 选中区域视图
                    const dt = elVisable - scrollViewWidth; // 可视差值
                    if(dt>0){
                        // 移动scroll-view
                        this.scrollLeft += dt;
                    }
                });

                // 定位选中值
                setTimeout(() => {
                    this.columPicked = new Array(this.defaultSelector.length);
                    this.columPicked.splice(i,1,this.defaultSelector[i]);
                }, 0);
            },
            // 标识滑动开始，只有微信小程序才有这样的事件
            pickstart() {
                // #ifdef MP-WEIXIN
                this.moving = true;
                // #endif
            },
            // 标识滑动结束
            pickend() {
                // #ifdef MP-WEIXIN
                this.moving = false;
                // #endif
            },
            init() {
                this.searchValue = '';
                this.setColumnNum();
                this.setDefaultSelector();
                this.setColumnData();
                // this.setSelectValue();
                this.setDefaultSelected();
            },
            // 获取默认选中列下标
            setDefaultSelector() {
                // 如果没有传入默认选中的值，生成长度为columnNum，用0填充的数组
                this.defaultSelector = this.defaultValue.length == this.columnNum ? this.defaultValue : Array(this.columnNum).fill(0);
                this.lastSelectIndex = this.$u.deepClone(this.defaultSelector);

                // 设置默认picked
                this.columPicked = this.$u.deepClone(this.defaultSelector);
            },
            /**
             * 设置默认选中
             */
            setDefaultSelected(){
                const columIndex = this.defaultValue.length == this.listLength? this.defaultValue: Array(this.listLength).fill(0);
                this.columnList = columIndex;

                // 设置scroll默认值
                this.selectValue = this.$u.deepClone(this.defaultSelectValue);
                // 重置第一列
                this.selectColumnIndex = 0;
            },
            // 计算列数
            setColumnNum() {
                // 通过父组件直接获取列数，自动计算仅判断第一个元素存在缺陷
                this.columnNum = this.listLength;
            },
            // 获取需要展示在picker中的列数据
            setColumnData() {
                let data = [];
                // this.selectValue = [];
                // 获得所有数据中的第一个元素
                let column = this.list[this.defaultSelector.length ? this.defaultSelector[0] : 0];
                // 通过循环所有的列数，再根据设定列的数组，得出当前需要渲染的整个列数组
                for (let i = 0; i < this.columnNum; i++) {
                    // 第一列默认为整个list数组
                    if (i == 0) {
                        data[i] = this.list;
                        column = column[this.childName];
                    } else {
                        // 大于第一列时，判断是否有默认选中的，如果没有就用该列的第一项
                        if(column){
                            // 如果第一项有child
                            data[i] = column;
                            column = column[this.defaultSelector[i]][this.childName];
                        }else{
                            // 没有child自动填充
                            data[i] = '';
                            column = '';
                        }
                    }
                }
                this.columnData = data;
            },
            // 获取默认选中的值，如果没有设置defaultValue，就默认选中每列的第一个
            setSelectValue() {
                let tmp = null;
                for(let i = 0; i < this.columnNum; i++) {
                    tmp = this.columnData[i][this.defaultSelector[i]];
                    let data = {
                        value: tmp ? tmp[this.valueName] : null,
                        label: tmp ? tmp[this.labelName] : null
                    };
                    // 判断是否存在额外的参数，如果存在，就返回
                    if(tmp && tmp.extra) data.extra = tmp.extra;
                    this.selectValue.push(data);
                }
            },
            // 列选项
            columnChange(e) {
                let index = null;
                // 根据当前选中index更新
                // const indexChanged = e;
                // if(this.selectColumnIndex != 0){
                //     columnIndex = this.lastSelectIndex.map((val, idx) => {
                //         if (idx == this.selectColumnIndex) val = indexChanged[idx];
                //     });
                // }
                index = this.selectColumnIndex;
                let columnIndex = e;
                // 由于后面是需要push进数组的，所以需要先清空数组
                this.selectValue = [];
                // 对比前后两个数组，寻找变更的是哪一列，如果某一个元素不同，即可判定该列发生了变化
                // this.lastSelectIndex.map((val, idx) => {
                //     if (val != columnIndex[idx]) index = idx;
                // });
                // this.defaultSelector.splice(this.selectColumnIndex,1,e[this.selectColumnIndex]);
                this.defaultSelector = columnIndex;
                for (let i = index + 1; i < this.columnNum; i++) {
                    // 当前变化列的下一列的数据，需要获取上一列的数据，同时需要指定是上一列的第几个的children，再往后的
                    // 获取当前选择数据
                    const list_data = this.columnData[i - 1][i - 1 == index ? columnIndex[index] : 0];
                    // 判断是否存在下列数据
                    if(list_data && list_data.hasOwnProperty(this.childName)){
                        // 存在下列数据
                        this.columnData[i] = list_data[this.childName];
                    }else{
                        // 默认是队列的第一个为默认选项
                        // 当本列数据不存在时，下一列必定不存在
                        // 对本列和下列置空
                        this.columnData[i] = [];
                        if(i < this.columnNum){
                            for(let j = i;j<this.columnNum;j++){
                                this.columnData[j] = [];
                            }
                        }
                        this.defaultSelector[i] = 0;
                        break; // 退出循环
                    }
                    // 改变的列之后的所有列，默认选中第一个
                    this.defaultSelector[i] = 0;
                }
                // 在历遍的过程中，可能由于上一步修改this.columnData，导致产生连锁反应，程序触发columnChange，会有多次调用
                // 只有在最后一次数据稳定后的结果是正确的，此前的历遍中，可能会产生undefined，故需要判断
                columnIndex.map((item, index) => {
                    if(index <= this.selectColumnIndex){
                        let data = this.columnData[index][columnIndex[index]];
                        let tmp = {
                            value: data ? data[this.valueName] : null,
                            label: data ? data[this.labelName] : null,
                        };
                        // 判断是否有需要额外携带的参数
                        if(data && data.extra !== undefined) tmp.extra = data.extra;
                        this.selectValue.push(tmp);
                    }
                });
                // 下列存在可选择时进行提示
                if(this.selectValue.length < this.listLength && this.columnData[this.selectValue.length].length){
                    this.selectValue.push({
                        value:'',
                        label:'请选择'
                    });
                }
                // 保存这一次的结果，用于下次列发生变化时作比较
                this.lastSelectIndex = columnIndex;
            },
            close() {
                this.$emit('input', false);
            },
            // 点击确定或者取消
            getResult(event = null) {
                // #ifdef MP-WEIXIN
                if (this.moving) return;
                // #endif
                // 有效性验证，存在不完整选择不进行确认
                let f = true;
                for(const v of this.selectValue){
                    if(!v.value) f = false;
                }
                if (event && f) {
                    this.$emit(event, this.selectValue);
                    this.$emit('updateDefault', this.lastSelectIndex);
                }
                this.close();
            },
            selectHandler() {
                this.$emit('click');
            },
            /**
             * 搜索 目前版本未验证和修改，如需使用可自行修改后实现数据搜索和服务端查询
             */
            searchOptions(e){
                if(this.timer){
                    clearTimeout(this.timer);
                }
                this.timer = setTimeout(()=>{
                    const value = e.target.value;
                    this.searchValue = value;
                    this.searchList = [];
                    let list = [];
                    new Promise(resolve =>{
                        if(this.serveSearch.f && value){
                            // server Search
                            const params = {
                                Subaction:this.serveSearch.params.subaction,
                                data:JSON.stringify({
                                    [this.serveSearch.params.data.serveValueKeys]:this.searchValue,
                                    MaxResults:this.serveSearch.params.data.MaxResults
                                })
                            };
                            const _this = this;
			    // 无用，报错的话删掉
                            uni.request({
                                url: '/mobile.pl?',
                                method: 'POST',
                                data: params,
                                header: {
                                    'Content-Type':'application/x-www-form-urlencoded;charset=UTF-8'
                                },
                                success(res) {
                                    if (res.data.result == '-100') {
                                        return false;
                                    }else{
                                        // list = $api.handleOptions(res.data.data);
                                        resolve();
                                    }
                                },
                                fail(err) {
                                    //请求失败
                                }
                            });
                        }else{
                            this.columnData = [JSON.parse(JSON.stringify(this.list))];
                            for(const group of this.columnData) {
                                for(const item of group){
                                    if(item.label.includes(value)){
                                        this.searchList.push(item.label);
                                        list.push(item);
                                    }
                                }
                            }
                            resolve();
                        }
                    }).then(()=>{
                        // 搜索后设置默认选中为搜索后的首项
                        this.selectValue = [];
                        if(list.length > 0){
                            this.selectValue.push({
                                value: list[0].value || null,
                                label: list[0].label || null
                            });
                        }
                        this.columnData = [JSON.parse(JSON.stringify(list))];
                    });
                },300);
            },
            onSubscribe(){
                // 监听键盘高度
                uni.onKeyboardHeightChange(res => {
                    this.keyBoardHieght = `${res.height}px`;
                    // 减小选择器高度 500rpx 是组件默认高度
                    this.selectCont_Height = '500rpx';
                });
            },
            offSubscribe(){
                // blur 恢复高度，取消订阅，节省内存
                this.keyBoardHieght = '0px';
                // 恢复选择器高度
                this.selectCont_Height = '920rpx';
                uni.offKeyboardHeightChange(res =>{
                });
            }
        }
    };
</script>

<style scoped lang="scss">

.u-select {

	&__action {
		position: relative;
		line-height: $u-form-item-height;
		height: $u-form-item-height;

		&__icon {
			position: absolute;
			right: 20rpx;
			top: 50%;
			transition: transform .4s;
			transform: translateY(-50%);
			z-index: 1;

			&--reverse {
				transform: rotate(-180deg) translateY(50%);
			}
		}
	}

	&__hader {
		&__title {
			color: $u-content-color;
		}
	}

	&--border {
		border-radius: 6rpx;
		border-radius: 4px;
		border: 1px solid $u-form-item-border-color;
	}

	&__header {
		// @include vue-flex;
        display: flex;
		align-items: center;
		justify-content: space-between;
		height: 80rpx;
		padding: 0 40rpx;
	}

	&__body {
		width: 100%;
		// height: 920rpx; // selct 加高处理
		overflow: hidden;
		background-color: #fff;

		&__picker-view {
			height: 100%;
			box-sizing: border-box;

			&__item {
				// @include vue-flex;
                display: flex;
				align-items: center;
				justify-content: center;
				font-size: 32rpx;
				color: $u-main-color;
				padding: 0 8rpx;

                // 选中的颜色
                .selected-item{
                    color: #2979FF;
                }
			}
		}
	}

    // 自定义搜索
    .select-search-cont{
        display: flex;
        border-bottom: 1rpx solid #b0b0b0;
        input{
            margin-left: 16rpx;
            font-size: 28rpx;
            color: #303133;
            &::-webkit-input-placeholder{
                color: #8F9399;
            }
        }
    }
    // 级联选择值
    .cascader-selected-value{
        padding: 0 20rpx;
        margin-bottom: 10rpx;
        &-scoll-view{
            white-space: nowrap;
        }
        &-item{
            display: inline-block;
            position: relative;
            margin-right: 20rpx;
            &-active{
                &::after{
                    content: '';
                    position: absolute;
                    bottom: 5rpx;
                    width: 40rpx;
                    left: 50%;
                    transform: translateX(-50%);
                    height: 5rpx;
                    background-color: #2979FF;
                    border-radius: 6rpx;
                }
            }
        }
    }
}
</style>
