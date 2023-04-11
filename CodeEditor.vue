<template>
  <codemirror
    ref="editor"
    :value="content"
    :options="codemirrorOptions"
    @input="handleInputChange"
    @blur="handleBlur"
  />
</template>

<script>
import { codemirror } from "vue-codemirror";
import "codemirror/lib/codemirror.css";

import "codemirror/mode/css/css.js";
import "codemirror/mode/htmlmixed/htmlmixed.js";
import "codemirror/mode/javascript/javascript.js";

// 主题
import "codemirror/theme/idea.css";

// #region 搜索功能
// find：Ctrl-F (PC), Cmd-F (Mac)
// findNext：Ctrl-G (PC), Cmd-G (Mac)
// findPrev：Shift-Ctrl-G (PC), Shift-Cmd-G (Mac)
// replace：Shift-Ctrl-F (PC), Cmd-Alt-F (Mac)
// replaceAll：Shift-Ctrl-R (PC), Shift-Cmd-Alt-F (Mac)
import "codemirror/addon/dialog/dialog.css";
import "codemirror/addon/dialog/dialog";
import "codemirror/addon/search/searchcursor";
import "codemirror/addon/search/search";
import "codemirror/addon/search/jump-to-line";
import "codemirror/addon/search/matchesonscrollbar";
import "codemirror/addon/search/match-highlighter";
// #endregion

// #region 代码提示功能
// 具体语言可以从 codemirror/addon/hint/ 下引入多个
import "codemirror/addon/hint/show-hint.css";
import "codemirror/addon/hint/show-hint.js";
import "codemirror/addon/hint/css-hint.js";
import "codemirror/addon/hint/html-hint.js";
import "codemirror/addon/hint/javascript-hint.js";
// #endregion

// #region 高亮行功能
import "codemirror/addon/selection/active-line";
import "codemirror/addon/selection/selection-pointer";
// #endregion

// #region 覆盖scrollbar样式功能
import "codemirror/addon/scroll/simplescrollbars.css";
import "codemirror/addon/scroll/simplescrollbars";
// #endregion

// 全屏功能 由于项目复杂，自带的全屏功能一般不好使
// import 'codemirror/addon/display/fullscreen.css'
// import 'codemirror/addon/display/fullscreen.js'

// 显示自动刷新
import "codemirror/addon/display/autorefresh.js";

// 括号匹配
import "codemirror/addon/edit/matchbrackets.js";
import "codemirror/addon/edit/closebrackets.js";

// 自动插入结束标签
// import "codemirror/addon/edit/matchtags.js";
// import "codemirror/addon/edit/closetag.js";

// 多语言支持？
// import 'codemirror/addon/mode/overlay'
// import 'codemirror/addon/mode/multiplex'

// 拼写提示
// import "codemirror/addon/lint/lint.js";
// import "codemirror/addon/lint/lint.css";
// import "codemirror/addon/lint/javascript-lint.js";
// import "codemirror/addon/lint/css-lint.js";

// #region  代码段折叠功能
import "codemirror/addon/fold/foldcode.js";
import "codemirror/addon/fold/foldgutter.js";
import "codemirror/addon/fold/foldgutter.css";
import "codemirror/addon/fold/brace-fold.js"; // 括号折叠
import "codemirror/addon/fold/comment-fold.js";
import "codemirror/addon/fold/indent-fold.js"; // 缩进折叠
import "codemirror/addon/fold/comment-fold.js";
// import 'codemirror/addon/fold/xml-fold.js'
// import 'codemirror/addon/fold/markdown-fold.js'
// #endregion

// #region  merge功能
// import 'codemirror/addon/merge/merge.css'
// import 'codemirror/addon/merge/merge.js'
// #endregion

import beautify from "js-beautify";

export default {
  name: "CodeEditor",

  // #region 组件基础
  components: { codemirror },
  props: {
    value: {
      type: String,
      required: true,
    },
    mode: {
      type: String,
      default: "javascript",
    },
  },
  // #endregion

  // #region 数据相关
  data() {
    return {
      content: "",
      codemirrorOptions: {
        // 语言及语法模式
        mode: this.mode,
        // 主题
        theme: "idea",
        // 显示函数
        line: false,
        // 显示行号
        lineNumbers: true,
        // 软换行
        lineWrapping: true,
        // tab宽度
        tabSize: 2,
        indentWithTabs: true,
        // 允许拖入的文件类型
        allowDropFileTypes: [this.mode],
        cursorScrollMargin: 1,
        // 高亮行功能
        styleActiveLine: true,
        // 调整scrollbar样式功能
        // scrollbarStyle: 'overlay',
        // 自动括号匹配功能
        matchBrackets: true, //括号匹配
        autoCloseBrackets: true, // 在键入时将自动关闭括号和引号
        // matchTags: { bothTags: true }, //标签匹配
        // autoCloseTags: true, // 在键入时将自动关闭标签
        autofocus: true,
        autoRefresh: true,
        // #region 代码折叠
        foldGutter: true,
        // foldOptions: { scanUp: true },
        gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter"],
        // #endregion
        autocorrect: true, // 自动更正
        spellcheck: true, // 拼写检查
        lint: false, // 检查格式
        showHint: true,
        hintOptions: {
          // 避免由于提示列表只有一个提示信息时，自动填充
          completeSingle: false,
        },
        extraKeys: { Tab: "autocomplete" },
        cursorHeight: 1,
      },
      formatOptions: {
        eol: "\n", //all.行结束符
        end_with_newline: false, //all,确保在文件结束时换行
        indent_char: "\t", //all,缩进字符，默认" "，\t 跳格(移至下一列)
        indent_size: 2, //all,缩进尺寸，默认4
        indent_with_tabs: true, //all,使用tab缩进,将会覆盖“indent_size”和“indent_char”设置，默认false
        preserve_newlines: true, //all,保留空行，默认“true”（保留换行符）
        max_preserve_newlines: null, //html、js,保留连续换行符个数，默认10（比如设为2，则会将2行以上的空行删除为只保留1行）
        wrap_line_length: 0, //html、js,在 n 个字符之后换行(设置为0忽略换行)
        html: {
          extra_liners: ["head", "body", "/html"], //配置标签列表，需要在这些标签前面额外加一空白行（换行符）
          indent_body_inner_html: false, //缩进 < body> 中的元素，默认：true
          indent_handlebars: true, //格式和缩进，默认为false
          indent_head_inner_html: false, //缩进 < head> 中的元素，默认：true
          indent_inner_html: true, // 缩进< head>和< body>中的元素（head和body也会缩进），默认：false
          indent_scripts: "normal", //缩进< script> 标签里的代码，有三个值：“keep”(对齐< script>标签)、“separate”(对齐左边界)、“normal”(正常缩进)，默认：“normal”
          inline: [], //各种标签的缩进[ "a", "abbr", "area", "audio", "b", "bdi", "bdo", "br", "button", "canvas", "cite", "code", "data", "datalist", "del", "dfn", "em", "embed", "i", "iframe", "img", "input", "ins", "kbd", "keygen", "label", "map", "mark", "math", "meter", "noscript", "object", "output", "progress", "q", "ruby", "s", "samp", "select", "small", "span", "strong", "sub", "sup", "svg", "template", "textarea", "time", "u", "var", "video", "wbr", "text", "acronym", "address", "big", "dt", "ins", "strike", "tt" ]
          wrap_attributes: "auto", //将属性换到新行，有5个值：“auto”(不换行)、“force”(第2个起换行)、 “force-aligned”(第2个起换行，与第1个属性对齐)、 “force-expand-multiline(所有属性都换行)，默认：“auto”
          wrap_attributes_indent_size: 4, //属性换行缩进大小,默认：indent_size
          unformatted: [], //数组中的标签不会重新格式化，默认：[]
          content_unformatted: [], //数组中标签的内容不会重新格式化，默认：[“pre”,“textarea”]
          void_elements: [], //定义自关闭标签,["area", "base", "br", "col", "embed", "hr", "img", "input", "keygen", "link", "menuitem", "meta", "param", "source", "track", "wbr", "!doctype", "?xml", "?php", "?=", "basefont", "isindex"]
        },
        js: {
          brace_style: "collapse", //括号风格，“collapse-preserve-inline”, “collapse”, “expand”, “end-expand”, or “none” ,默认“collapse”
          break_chained_methods: true, //中断多行间的链式方法调用，默认true
          comma_first: false, //将逗号放在新行的开头而不是结尾，默认为false
          e4x: true, //	不受影响地传递E4X xml文本,默认为false
          indent_level: 0, //	初始缩进级别
          jslint_happy: true, //	开启jslint-stricter的严格模式（强制开启“space_after_anon_function”选项）,默认false
          keep_array_indentation: true, //	保持数组缩进，默认false
          keep_function_indentation: true, //	保持函数缩进，默认false
          operator_position: "before-newline", //	将操作符移动到新行之前或之后，或保持原样。“before-newline”， “after-newline”，或"preserve-newline",默认为"before-newline"
          space_after_anon_function: true, //	在匿名函数的括号前加空格,默认为false
          space_after_named_function: true, //	在具名函数之前加一个空格
          space_before_conditional: true, //	在条件语句之前保留一个空格，默认true
          space_in_empty_paren: true, //	在括号中留空格，默认为false
          space_in_paren: true, //	在函数参数之间增加空格，默认为false
          unescape_strings: true, //	解码用xNN表示法编码的可打印字符,默认为false
          unindent_chained_methods: true, //	Unindent链接方法,默认为false
        },
      },
    };
  },
  watch: {
    value(newVal) {
      this.content = newVal;
    },
    mode(newVal) {
      this.$refs.editor.options.mode = newVal;
    },
  },
  methods: {
    handleInputChange(value) {
      this.$emit("input", value);
    },
    handleBlur() {
      if (this.mode == "css")
        this.content = beautify.css_beautify(this.content || "", {
          indent_size: 2,
        });
      else this.content = beautify(this.content || "", this.formatOptions);
    },
  },
};
</script>
<style lang="less">
.CodeMirror-hints {
  z-index: 1100 !important;
}
</style>