# NeoBinding
Android View Model binding framework, Simple but powerful.

## Usage

UI Component

    class MainActivityUI : ViewBinderComponent<MainActivity> {
        override fun builder(): AnkoContext<*>.() -> Unit = {
            verticalLayout {
                verticalLayout {
                    backgroundColor = Color.WHITE
                    leftPadding = dip(14)
                    editText {
                        hint = "请输入手机号或者电子邮箱地址"
                        bind(text(path="name", mode = BindingMode.TwoWay))
                    }.lparams(width = matchParent)
                    view { backgroundResource = R.color.color_f2 }.lparams(width = matchParent, height = 1)
                    editText {
                        hint = "请输入密码"
                        bind(text(path="password", mode = BindingMode.TwoWay))
                    }.lparams(width = matchParent)
                }.lparams(width = matchParent)
                textView {
                    text = "登录"
                    textSizeDimen = R.dimen.font_38
                    textColor = Color.WHITE
                    verticalPadding = dip(10.4f)
                    isClickable = true
                    bind(click("login"))
                    bind(enabled(paths=listOf("name", "password"), converter = ArrayToBooleanConverter()))
                }.lparams(width = matchParent) { margin = dip(14) }.let { it.gravity = Gravity.CENTER }
            }
        }
    }
    
ViewModel

    class LoginViewModel(val delegate: LoginViewModel.LoginDelegate) : ViewModel<String>() {

        var level: Int by Delegates.bindProperty("level", 3)
        var name: String by Delegates.bindProperty("name", "xxxxxxx@xxxxx.com")
        var password: String by Delegates.bindProperty("password", "xxxxxxxxx")

        val login: Command<String> by Delegates.bindCommand("login", { if(name.equals("wangbin")) "SUCCESS" else throw RuntimeException("Incorrect name or password") }, { t: String -> delegate.onLoginSuccess(t)}, { e -> delegate.onLoginFailed(e)})

        override fun notifyPropertyChange(t: String?) {
        }

        interface LoginDelegate {
            fun onLoginSuccess(s: String)
            fun onLoginFailed(e: Throwable)
        }
    }
    
Binding

    MainActivityUI().setContentView(activity).bindTo(bindingContext, LoginViewModel(delegate object))
    
## Extend Binding Property 

Event
    
    fun View.click(path: String) : PropertyBinding {
        return BindingAssembler.commandBinding(path, clicks(), enabled())
    }

Property

    fun View.enabled(path: String, mode: BindingMode, converter: Any? = EmptyOneWayConverter<Any>()) : PropertyBinding =             when(mode) {
        BindingMode.OneWay -> BindingAssembler.oneWayPropertyBinding<Boolean, Any>(path, enabled(), converter as? OneWayConverter<Boolean>)
        BindingMode.OneWayToSource -> throw UnsupportedOperationException("OneWayToSource not allowed for enabled")
        BindingMode.TwoWay -> throw UnsupportedOperationException("TwoWay not allowed for enabled")
    }
    fun View.enabled(paths: List<String>, converter: MultipleConverter<Boolean>) : PropertyBinding = BindingAssembler.multiplePropertyBinding(paths, enabled(), converter)

    fun TextView.text(path: String, mode: BindingMode, converter: Any? = EmptyOneWayConverter<Any>()) : PropertyBinding = when(mode) {
        BindingMode.OneWay -> BindingAssembler.oneWayPropertyBinding<CharSequence, Any>(path, text(), converter as? OneWayConverter<CharSequence>)
        BindingMode.OneWayToSource -> BindingAssembler.oneWayPropertyBinding<String, Any>(path, textChanges().map { it.toString() }.skip(1), converter as? OneWayConverter<Any>)
        BindingMode.TwoWay -> BindingAssembler.twoWayPropertyBinding(path, textChanges().map { it.toString() }.skip(1), text(), converter as? TwoWayConverter<String, String>)
    }
    fun TextView.text(paths: List<String>, converter: MultipleConverter<CharSequence>) : PropertyBinding = BindingAssembler.multiplePropertyBinding(paths, text(), converter)
