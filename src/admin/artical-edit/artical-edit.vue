<template>
  <div class="detailtWrapper" @click.stop="showList = false">
    <div class="blogEdit">
      <h3 class="title">添加文章</h3>
      <div class="icon" @click="closeEditor"><span class="icon-closeAll"></span></div>
    </div>
    <div class="content">
      <div class="con-item titleBox">
        <div class="label">
          <label>文章标题</label>
        </div>
        <div class="input titleInput">
          <input type="text" name="title" class="title" v-model="title" placeholder="请输入文章标题">
        </div>
      </div>
      <div class="con-item modelBox">
        <div class="default">
          <div class="label">
            <label>文章分类</label>
          </div>
          <div class="input modelInput" @click.stop="chooseModel">
            <input type="text" name="model" class="model" v-model="model" value="javaScript" readonly>
            <span ref="circle" class="icon-circle"></span>
            <div class="optionList" v-show="showList === true">
              <option-list :options="options" :show="showList" @clickoption="clickoption"></option-list>
            </div>
          </div>
        </div>
        <div class="manageLabel">
          <button type="button" @click.stop="showLabelList">管理分类</button>
        </div>
      </div>
      <div class="con-item tagsBox">
        <div class="label">
          <label>标签</label>
        </div>
        <div class="input tagsInput">
          <input type="text" name="tags" class="tags" v-model="tags" placeholder="请输入标签，以‘/’分割">
          <ul class="tagHistory">
            <li v-for="item in tagsHistory" @click="selectTag(item.tag_text)">{{item.tag_text}}</li>
          </ul>
        </div>
      </div>
      <div class="con-item summaryBox">
        <div class="label">
          <label>内容摘要</label>
        </div>
        <div class="input summaryInput">
          <textarea name="summary" class="summary" v-model="summary" placeholder="请输入文章摘要"></textarea>
        </div>
      </div>
      <div class="con-item FileBox">
        <div class="label">
          <label for="photo">上传图片</label>
        </div>
        <div class="input fileInput">
          <input type="file" ref="upload" name="photo" id="photo" accept="image/*" @change="getImg" />
          <button type="button" @click="uploadImage" v-show="showUpload" class="uploadImg">上传</button>
        </div>
      </div>
      <div class="con-item showType">
        <div class="postNow">
          <div class="label">
            <label>立即发布</label>
          </div>
          <div class="input radios">
            <input type="radio" name="isShow" value="1" v-model="checked">
          </div>
        </div>
        <div class="saveDraft">
          <div class="label">
            <label>存草稿箱</label>
          </div>
          <div class="input radios">
            <input type="radio" name="isShow" value="0" v-model="checked">
          </div>
        </div>
      </div>
      <div class="con-item editorBox">
        <div class="label">
          <label>文章内容</label>
        </div>
        <div class="input editorInput">
          <textarea class="markdown_input" v-model="artical" @input="update" id="blog-content"></textarea>
          <div class="markdown_compiled" v-html="compiledMarkdown"></div>
        </div>
      </div>
      <div class="doneBtn">
        <button type="button" class="pub" @click.stop="saveArtical">发布</button>
      </div>
    </div>
    <attention :text="attText" :isOK="attIcon" ref="attBox"></attention>
    <div class="labelWrapper">
      <label-list class="labelList" :labelList="options" :showFlag="showFlag" @closeLabelList="closeLabelList" @deleteLabel='deleteLabel' @addLabel="addLabel"></label-list>
    </div>
  </div>
</template>

<script>
  import Attention from '../../base/attention/attention';
  import LabelList from '../../admin/labelList/labelList';
  import OptionList from '../../base/option-list/option-list';
  import {showAttentionMixin} from '../../common/js/mixin';
  import {saveBlog, getClassify, updateBlog, uploadImg} from '../../api/editor';
  import {getTags} from '../../api/archives';
  import {mapGetters, mapMutations} from 'vuex';
  import _ from 'lodash';
  import marked from 'marked';
  import hightlight from 'highlight.js';
  import '../../common/css/atom-one-light.css';
  marked.setOptions({
    hightlight: function (code) {
      return hightlight.hightlightAuto(code).value;
    }
  });
 
  export default {
    mixins: [showAttentionMixin],
    data () {
      return {
        title: '',
        summary: '',
        tags: '',
        artical: '',
        model: '',
        showList: false,
        options: [],
        checked: 0,
        showFlag: false,
        showUpload: false,
        tagsHistory: []
      };
    },
    computed: {
      // 编译Markdown
      compiledMarkdown () {
        return marked(this.artical, {sanitize: true});
      },
      ...mapGetters([
        'editBlog',
        'type'
      ])
    },
    created () {
      this.formData = new FormData();
      this.initBlog();
      this._getClassify();
      this.getTagsHistory(); 
    },
    methods: {
      selectTag (tag) {
        if (this.tags === '') {
          this.tags = tag;
        } else {
          this.tags += '/' + tag;
        }
      },
      getTagsHistory () {
        getTags().then(res => {
          if (res.status === 0) {
            this.tagsHistory = res.data;
          }
        });
      },
      getImg () {
        if (this.$refs.upload.files.length > 0) {
          this.showUpload = true;
          for (let i = 0; i < this.$refs.upload.files.length; i++) {
            this.formData.append('photo', this.$refs.upload.files[i]);
          }
        }
      },
      uploadImage () {
        uploadImg(this.formData).then(res => {
          if (res.status === 0) {
            this.artical += '![](' + res.url + ')';
          }
        });
      },
      // 延时赋值给content
      update: _.debounce(function (e) {
        this.artical = e.target.value;
      }, 300),
      saveArtical () {
        if (this.title === '') {
          this.showAttention('请输入文章标题', false);
        } else if (this.tags === '') {
          this.showAttention('请输入文章标签', false);
        } else if (this.summary === '') {
          this.showAttention('请输入文章摘要', false);
        } else if (this.artical === '') {
          this.showAttention('请输入文章内容', false);
        } else if (!this.type) {
          this._saveBlog();
        } else {
          this._updateBlog();
          this.setEditBlog({});
        }
      },
      initBlog () {
        if (this.type) {
          this.title = this.editBlog.blog_title;
          this.artical = this.editBlog.blog_content;
          this.summary = this.editBlog.blog_description;
          this.tags = this.editBlog.blog_tags;
          this.model = this.editBlog.classify_text;
          this.checked = this.editBlog.blog_isShow;
        }
      },
      chooseModel () {
        this.showList = !this.showList;
        if (this.showList) {
          this.$refs.circle.style.transform = 'rotate(180deg)';
        } else {
          this.$refs.circle.style.transform = '';
        }
      },
      clickoption (item) {
        this.model = item.text;
      },
      closeEditor () {
        this.$router.go(-1);
      },
      showLabelList () {
        this.showFlag = true;
      },
      closeLabelList () {
        this.showFlag = false;
      },
      deleteLabel (id) {
        let index = this.options.findIndex((item) => {
          return item.id === id;
        });
        this.options.splice(index, 1);
      },
      addLabel (item) {
        this.options.push(item);
      },
      routerGo () {
        if (!this.checked) {
          this.$router.push({path: '/admin/draft'});
        } else {
          this.$router.push({path: '/admin/article'});
        }
      },
      _saveBlog () {
        const blog = {
          title: this.title,
          classify_text: this.model,
          tags: this.tags,
          description: this.summary,
          content: this.artical,
          isShow: this.checked
        };
        saveBlog(blog).then(res => {
          if (res.status === 0) {
            this.showAttention(res.info, true);
            setTimeout(() => {
              this.routerGo();
            }, 4000);
          } else {
            this.showAttention(res.info, false);
          }
        });
      },
      _updateBlog () {
        const blog = {
          id: this.editBlog.blog_id,
          title: this.title,
          classify_text: this.model,
          tags: this.tags,
          description: this.summary,
          content: this.artical,
          isShow: this.checked
        };
        updateBlog(blog).then(res => {
          if (res.status === 0) {
            this.showAttention(res.info, true);
          } else {
            this.showAttention(res.info, false);
          }
        });
      },
      _getClassify () {
        getClassify().then(res => {
          let i = 0;
          for (i; i < res.data.length; i++) {
            this.options.push({id: res.data[i].classify_id, text: res.data[i].classify_text});
          };
        });  
      },
      ...mapMutations({
        setEditBlog: 'SET_EDITBLOG'
      })
    },
    components: {
      Attention,
      OptionList,
      LabelList
    }
  };
</script>

<style lang="less" rel="stylesheet/less">
  .detailtWrapper{
    position: relative;
    color: #333;
    padding-bottom: 20px;
    .blogEdit{
      display: flex;
      justify-content: space-between;
      height: 42px;
      line-height: 42px;
      border-bottom: 1px solid #eee;
      font-size: 14px;
      background-color: #F8F8F8;
      border-radius: 2px 2px 0 0;
      padding: 0 20px;
      .icon{
        height: 42px;
        line-height: 42px;
        cursor: pointer;
        &:hover{
          color: #ADADAD;
        }
      }
    }
    .content{
      margin: 10px 10px 0;
      .con-item{
        display: flex;
        margin-bottom: 15px;
        .default, .postNow, .saveDraft{
          display: flex;
        }
      }
      .label{
        padding: 9px 15px;
        width: 80px;
        font-weight: 400;
        text-align: right;
      }
      input{
        width: 100%;
        height: 38px;
        line-height: 38px;
        padding-left: 10px;
        border: 1px solid #e6e6e6;
        border-radius: 2px;
        outline: none;
        box-sizing: border-box;
      }
     .titleInput{
        width: 90%;
      }
      .tagsInput{
        width: 60%;
        .tagHistory{
          margin-top: 10px;
          width: 300px;
          li{
            display: inline-block;
            padding: 4px 8px;
            font-size: 13px;
            color: #fff;
            background: #ccc;
            margin: 4px;
            border-radius: 4px;
            cursor: pointer;
          }
        }
      }
      .manageLabel{
        margin-left: 20px;
        button{
          width: 100px;
          height: 38px;
          color: #333;
        }
      }
      .modelInput{
        position: relative;
        .icon-circle{
          position: absolute;
          right: 10px;
          top: 50%;
          margin-top: -8px;
          transition: all .2s;
        }
        .optionList{
          width: 100%;
          position: absolute;
          top: 44px;
          left: 0;
        }
      }
      .summaryInput{
        width: 90%;
        textarea{
          width: 99%;
          height: 100px;
          font-size: 16px;
          padding: 6px 10px;
          border: 1px solid #e6e6e6;
          resize: none;
          outline: none;
        }
      }
      .uploadImg{
        width: 80px;
        height: 26px;
        border: none;
        color: #fff;
        font-size: 16px;
        background-color: #006030;
        border-radius: 2px;
      }
      .editorInput{
        margin: 0;
        width: 100%;
        height: 500px;
        font-family: 'Helvetica Neue', Arial, sans-serif;
        color: #333;
        border: 1px solid #ccc;
        .markdown_input, .markdown_compiled{
          display: inline-block;
          width: 49%;
          height: 100%;
          vertical-align: top;
          box-sizing: border-box;
          padding: 20px;
        }
        .markdown_input{
          border: none;
          border-right: 1px solid #ccc;
          resize: none;
          outline: none;
          background-color: #f6f6f6;
          font-size: 14px;
          font-family: 'Monaco', courier, monospace;
        }
        .markdown_compiled{
          overflow: auto;
          word-wrap:break-word;
        }
      }
      .showType{
        width: 60%;
        .radios{
          input{
            width: 20px;
            height: 20px;
            margin-top: 8px;
          }
        }
      }
      .doneBtn{
        width: 100%;
        text-align: center;
        button{
          width: 100px;
          height: 35px;
          border: none;
          color: #fff;
          font-size: 16px;
          background-color: #006030;
          border-radius: 2px;
        }
      }
    }
    .labelWrapper{
      position: fixed;
      top: 25%;
      left: 30%;
      z-index: 9998;
    }
  }
  /* markdown css */
  .editorInput{
    font-size: 16px;
    line-height: 27px;
    p{
      margin-bottom: 25px;
    }
    ul{
      list-style: disc;
      margin: 16px 0;
      padding-left: 40px;
      li{
        list-style: disc;
      }
    }
    em{
      background: #eee;
      text-shadow: 0 1px #fff;
      padding: 0 0.3em;
      margin: 0 0.3em;
      font-family: inherit;
    }
    strong{
      font-weight: bold;
    }
    table {
        margin: 10px 0 15px 0;
        border-collapse: collapse;
    }
    th {
        text-align: center;
        padding: 5px 10px;  
        border-bottom: 2px solid #ddd;
    }
    td {
      padding: 8px;
      line-height: 1.42857143;
      vertical-align: top;
      border-top: 1px solid #ddd;
    }
    a {
        color: #0069d6;
    }
    a:hover {
        color: #0050a3;
        text-decoration: none;
    }
    a img {
        border: none;
    }
    h1,
    h2,
    h3,
    h4,
    h5,
    h6 {
        color: #404040;
        line-height: 36px;
    }
    h1 {
        margin-bottom: 25px;
        padding: 0.3em 0;
        font-size: 20px;
        font-weight: 600;
        line-height: 30px;
        border-bottom: 1px solid #eee;
    }
    h2 {
        font-size: 17px;
        font-weight: 600;
        line-height: 30px;
        margin-bottom: 25px;
    }
    h3 {
        font-size: 18px;
    }
    h4 {
        font-size: 16px;
    }
    h5 {
        font-size: 14px;
    }
    h6 {
        font-size: 13px;
    }
    hr {
        margin: 0 0 19px;
        border: 0;
        border-bottom: 1px solid #ccc;
    }
    blockquote {
        margin: 20px 0;
        padding: 15px;
        border-left: 3px solid #ccc;
        background-color: #f7f7f7;
        font-family: inherit;
    }
    blockquote p {
        font-size: 16px;
        font-weight: 300;
        line-height: 18px;
        margin-bottom: 0;
    }
    code, pre {
        font-family: Monaco, Andale Mono, Courier New, monospace;
    }
    code {
        background-color: #fee9cc;
        color: rgba(0, 0, 0, 0.75);
        padding: 1px 3px;
        font-size: 12px;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
        border-radius: 3px;
    }
    pre {
        margin-bottom: 25px;
        padding: 16px;
        overflow: auto;
        font-size: 85%;
        background-color: #f7f7f7;
        word-wrap: normal;
        border-radius: 3px;
        color: #333;
        font: 13px Consolas, "Liberation Mono", Menlo, Courier, monospace;
        font-family: inherit;
    }
    pre code {
        background: transparent;
        padding: 0
    }
    sup {
        font-size: 0.83em;
        vertical-align: super;
        line-height: 0;
    }
  }
</style>