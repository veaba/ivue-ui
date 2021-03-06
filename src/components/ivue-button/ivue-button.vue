<script>
import IvueButtonContent from "./ivue-button-content";
import IvueRouterLinkProps from "../../utils/mixins/ivue-router-link-props";
import Colorable from '../../utils/mixins/colorable';
import { inject as registrableInject } from '../../utils/mixins/registrable';
import ripple from '../../utils/directives/ripple';

const prefixCls = 'ivue-button';

export default {
    name: prefixCls,
    mixins: [
        Colorable,
        IvueRouterLinkProps,
        registrableInject('buttonGroup')
    ],
    directives: {
        ripple
    },
    data () {
        return {
            /*
            * 波纹效果激活
            *
            * @type {Boolean}
            */
            rippleActive: false,
            /*
            * 按钮是否激活状态
            *
            * @type {Boolean}
            */
            isActive: false
        }
    },
    props: {
        /*
        * 在按钮上创建一个锚点。在这种情况下，生成的标签将是 a
        *
        * @type {String}
        */
        href: {
            type: String,
            default: null
        },
        /*
        * 将类型应用于按钮 - 它不会影响链接
        *
        * @type {String}
        */
        type: {
            type: String,
            default: 'button'
        },
        /*
        * 禁用该按钮并阻止其操作
        *
        * @type {Boolean}
        */
        disabled: {
            type: Boolean,
            default: false
        },
        /*
        * 用于router-link 跳转
        *
        * @type {String}
        */
        to: null,
        /*
        * 激活涟漪效果
        *
        * @type {Boolean}
        */
        ripple: {
            type: Boolean,
            default: true
        },
        /*
        * 是否扁平按钮
        *
        * @type {Boolean}
        */
        flat: Boolean,
        /*
        * 凹陷的按钮依然保持其背景色，但没有框阴影
        *
        * @type {Boolean}
        */
        depressed: Boolean,
        /*
        * 圆形图标
        *
        * @type {Boolean}
        */
        icon: Boolean,
        /*
        * 轮廓按钮从当前色彩应用继承他们的边框颜色。
        *
        * @type {Boolean}
        */
        outline: Boolean,
        /**
         * 当使用中心选项时，纹波将始终来自目标的中心。
         *
         * @type {Boolean}
         */
        center: Boolean
    },
    mounted () {
        if (this.buttonGroup) {
            this.buttonGroup.register(this)
        }
    },
    beforeDestroy () {
        if (this.buttonGroup) {
            this.buttonGroup.unregister(this)
        }
    },
    computed: {
        // 是否显示涟漪效果
        rippleWorks () {
            return !this.disabled;
        },
        // 判断按钮是否激活
        activeButton () {
            if (this.isActive) {
                return `${prefixCls}--active`;
            }
        },
        classes () {
            return {
                [`${prefixCls}--raised`]: !this.flat,
                [`${prefixCls}--flat`]: this.flat,
                [`${prefixCls}--depressed`]: (this.depressed && !this.flat) || this.outline,
                [`${prefixCls}--icon`]: this.icon,
                [`${prefixCls}--outline`]: this.outline
            }
        },
        computedRipple () {
            if (this.ripple && this.center) {
                return {
                    center: true
                }
            }
            else if (this.ripple && !this.disabled) {
                return this.ripple;
            }

            return false;
        }
    },
    components: {
        IvueButtonContent
    },
    render (createElement) {
        const buttonContent = createElement(IvueButtonContent, {}, this.$slots.default);

        let buttonAttrs = {
            staticClass: prefixCls,
            class: {
                'isMobile': this.mobile,
                'ivue-button--active': this.isActive,
                ...this.classes
            },
            data: {
                mobile: false
            },
            attrs: {
                ...this.attrs,
                href: this.href,
                disabled: this.disabled,
                type: !this.href && (this.type || 'button')
            },
            directives: [
                {
                    name: 'ripple',
                    value: this.computedRipple
                }
            ],
            on: {}
        }

        if (!this.to) {
            buttonAttrs.on = {
                ...this.$listeners,
                touchstart: (event) => {
                    // 是否显示涟漪效果
                    if (this.rippleWorks) {
                        this.rippleActive = event;
                    }
                    // this.$listeners.touchstart && this.$listeners.touchstart(event);

                    this.mobile = true;
                },
                touchmove: (event) => {
                    // 是否显示涟漪效果
                    if (this.rippleWorks) {
                        this.rippleActive = event;
                    }
                    // this.$listeners.touchmove && this.$listeners.touchmove(event);

                },
                click: (event) => {
                    // 是否显示涟漪效果
                    if (this.rippleWorks) {
                        this.rippleActive = event;
                    }
                    // this.$listeners.click && this.$listeners.click(event);

                    this.$emit('click', this);
                }
            }
        }

        let _tag = 'button';

        // a 标签
        if (this.href) {
            _tag = 'a'
        }
        // 渲染成 router-link
        else if (this.to) {
            const { tag, data } = this.generateRouteLink();

            _tag = tag;

            buttonAttrs.props = this.$props;
        }

        // 设置颜色
        const setColor = (!this.outline && !this.flat) ? this.setBackgroundColor : this.setTextColor;

        return createElement(_tag, setColor(this.color, buttonAttrs), [buttonContent]);
    }
}
</script>
