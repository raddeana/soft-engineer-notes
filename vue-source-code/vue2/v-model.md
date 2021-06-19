### 指令的编译
#### 解析模版字符串生成AST
```javascript
const ast = parse(template.trim(), options)
```

#### 优化语法树
```javascript
optimize(ast, options)
```

#### 生成代码
```javascript
const code = generate(ast, options)
```

#### generate
genElement()->genData$2()->genDirectives()生成code，再把 code 用 with(this){return ${code}}} 包裹起来,最终的到render函数
```javascript
 function generate (ast, options) {
  var state = new CodegenState(options);
  var code = ast ? genElement(ast, state) : '_c("div")';
  return {
    render: ("with(this){return " + code + "}"),
    staticRenderFns: state.staticRenderFns
  }
}
```

#### model
```javascript
warn$1 = _warn;
var value = dir.value;
var modifiers = dir.modifiers;
var tag = el.tag;
var type = el.attrsMap.type;

{
  // inputs with type="file" are read only and setting the input's
  // value will throw an error.
  if (tag === 'input' && type === 'file') {
    warn$1(
      "<" + (el.tag) + " v-model=\"" + value + "\" type=\"file\">:\n" +
      "File inputs are read only. Use a v-on:change listener instead.",
      el.rawAttrsMap['v-model']
    );
  }
}

if (el.component) {
  genComponentModel(el, value, modifiers);
  // component v-model doesn't need extra runtime
  return false
} else if (tag === 'select') {
  genSelect(el, value, modifiers);
} else if (tag === 'input' && type === 'checkbox') {
  genCheckboxModel(el, value, modifiers);
} else if (tag === 'input' && type === 'radio') {
  genRadioModel(el, value, modifiers);
} else if (tag === 'input' || tag === 'textarea') {
  genDefaultModel(el, value, modifiers);
} else if (!config.isReservedTag(tag)) {
  genComponentModel(el, value, modifiers);
  // component v-model doesn't need extra runtime
  return false
} else {
  warn$1(
    "<" + (el.tag) + " v-model=\"" + value + "\">: " +
    "v-model is not supported on this element type. " +
    'If you are working with contenteditable, it\'s recommended to ' +
    'wrap a library dedicated for that purpose inside a custom component.',
    el.rawAttrsMap['v-model']
  );
}

// ensure runtime directive metadata
return true
```

#### genDefaultModel
```javascript
var event = lazy ? 'change' : type === 'range' ? RANGE_TOKEN : 'input';

var code = genAssignmentCode(value, valueExpression);

if (needCompositionGuard) {
  code = "if($event.target.composing)return;" + code;
}

addProp(el, 'value', ("(" + value + ")"));
addHandler(el, event, code, null, true);
if (trim || number) {
  addHandler(el, 'blur', '$forceUpdate()');
}
```

#### genAssignmentCode
```javascript
function genAssignmentCode (value, assignment) {
  var res = parseModel(value);
  if (res.key === null) {
    return (value + "=" + assignment)
  } else {
    return ("$set(" + (res.exp) + ", " + (res.key) + ", " + assignment + ")")
  }
}
```

#### $event.target.composing
用于判断此次input事件是否是IME构成触发的，如果是IME构成，直接return；IME 是输入法编辑器(Input Method Editor) 的英文缩写，IME构成指我们在输入文字时，处于未确认状态的文字

### 用户输入事件的绑定和触发vmodel更新
#### inserted
```javascript
    if (vnode.tag === 'select') {
      // #6903
      if (oldVnode.elm && !oldVnode.elm._vOptions) {
        mergeVNodeHook(vnode, 'postpatch', () => {
          directive.componentUpdated(el, binding, vnode)
        })
      } else {
        setSelected(el, binding, vnode.context)
      }
      el._vOptions = [].map.call(el.options, getValue)
    } else if (vnode.tag === 'textarea' || isTextInputType(el.type)) {
      el._vModifiers = binding.modifiers
      if (!binding.modifiers.lazy) {
        el.addEventListener('compositionstart', onCompositionStart)
        el.addEventListener('compositionend', onCompositionEnd)
        // Safari < 10.2 & UIWebView doesn't fire compositionend when
        // switching focus before confirming composition choice
        // this also fixes the issue where some browsers e.g. iOS Chrome
        // fires "change" instead of "input" on autocomplete.
        el.addEventListener('change', onCompositionEnd)
        /* istanbul ignore if */
        if (isIE9) {
          el.vmodel = true
        }
      }
    }
```
#### trggier
```javascript
  function onCompositionStart (e) {
    e.target.composing = true
  }

  function onCompositionEnd (e) {
    // prevent triggering an input event for no reason
    if (!e.target.composing) return
    e.target.composing = false
    trigger(e.target, 'input')
  }

  function trigger (el, type) {
    const e = document.createEvent('HTMLEvents')
    e.initEvent(type, true, true)
    el.dispatchEvent(e)
  }
```
