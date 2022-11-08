<template>
  <div>
      <textarea :id="id">{{ content }}</textarea>
  </div>
</template>

<script>

    export default {
        name: 'tinymce',
        props: { 
                id : {
                    type : String,
                    required : true
                },
                htmlClass : { default : '', type : String},
                value : { default : '' },
                plugins : { default : function(){ 
                                    return [
                                        'advlist autolink lists link image charmap print preview hr anchor pagebreak',
                                        'searchreplace wordcount visualblocks visualchars code fullscreen',
                                        'insertdatetime media nonbreaking save table contextmenu directionality',
                                        'template paste textcolor colorpicker textpattern imagetools toc help emoticons hr codesample'
                                    ];
                                } , type: Array
                            },
                toolbar1: { default :'formatselect | bold italic  strikethrough  forecolor backcolor | link | alignleft aligncenter alignright alignjustify  | numlist bullist outdent indent  | removeformat', type: String},
                toolbar2: { default : '', type: String },
                other_options: { default : function() { return {}; }, type: Object},
                readonly: { default: false, type: Boolean }
        },
        data(){
            return {
                content : '',
                checkerTimeout: null,
                isTyping : false
            }; 
        },
        mounted(){
            this.content = this.value;
            this.importTinyMCE()
            .then(() => {
                this.init();  
            })
        },
        beforeDestroy () {
            this.editor.destroy();
        },
        watch: {
            value : function (newValue){
                if(!this.isTyping){
                    if(this.editor !== null)
                        this.editor.setContent(newValue);
                    else
                        this.content = newValue;
                }
            },
            readonly(value){
                if(value){
                    this.editor.setMode('readonly');
                } else {
                    this.editor.setMode('design');
                }
            }
        },
        computed: {
            editor() {
                return window.tinymce.getInstanceById ? 
                    window.tinymce.getInstanceById(this.id) : 
                    (window.tinymce.get ? window.tinymce.get(this.id) : window.tinymce.editors[this.id])
            }
        },
        methods: {
            init(){
                let options = {
                    selector: '#' + this.id,
                    skin: false,
                    toolbar1: this.toolbar1,
                    toolbar2: this.toolbar2,
                    plugins: this.plugins,
                    init_instance_callback : this.initEditor
                };
                window.tinymce.init(this.concatAssciativeArrays(options, this.other_options));
            },
            initEditor(editor) {
                editor.on('KeyUp', (e) => {
                    this.submitNewContent(editor);
                });
                editor.on('Change', (e) => {
                    if(editor.getContent() !== this.value){
                        this.submitNewContent(editor);
                    }
                    this.$emit('editorChange', e);
                });
                editor.on('init', (e) => {
                    editor.setContent(this.content);
                    this.$emit('input', this.content);
                });
                if(this.readonly){
                    editor.setMode('readonly');
                } else {
                    editor.setMode('design');
                }

                this.$emit('editorInit', editor);
            },
            concatAssciativeArrays(array1, array2){
                if(array2.length === 0) return array1;
                if(array1.length === 0) return array2;
                let dest = [];
                for ( let key in array1) dest[key] = array1[key];
                for ( let key in array2) dest[key] = array2[key];
                return dest;
            },
            submitNewContent(editor){
                this.isTyping = true;
                if(this.checkerTimeout !== null)
                    clearTimeout(this.checkerTimeout);
                    this.checkerTimeout = setTimeout(()=>{
                        this.isTyping = false;
                    }, 300);

                this.$emit('input', editor.getContent());
            },

            importTinyMCE() {
                if (!window.tinymce) {
                    return import(/* webpackMode: "eager" */'tinymce/tinymce')
                        .then(() => {
                            const promises = [];

                            promises.push(import(/* webpackMode: "eager" */'tinymce/themes/modern/theme'));

                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/advlist'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/wordcount'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/autolink'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/autosave'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/charmap'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/codesample'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/contextmenu'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/emoticons'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/fullscreen'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/hr'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/imagetools'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/insertdatetime'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/link'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/media'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/noneditable'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/paste'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/print'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/searchreplace'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/tabfocus'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/template'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/textpattern'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/visualblocks'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/anchor'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/autoresize'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/bbcode'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/code'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/colorpicker'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/directionality'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/fullpage'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/help'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/image'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/importcss'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/legacyoutput'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/lists'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/nonbreaking'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/pagebreak'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/preview'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/save'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/spellchecker'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/table'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/textcolor'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/toc'));
                            promises.push(import(/* webpackMode: "eager" */'tinymce/plugins/visualchars'));
                            
                            promises.push(import(/* webpackMode: "eager" */'tinymce/skins/lightgray/skin.min.css'));

                            return Promise.all(promises)
                        })
                        .catch((e) => {
                            console.log('Error!', e)
                        })
                } else {
                    return new Promise((resolve) => {
                        resolve()
                    })
                }
            }
        }
    }
</script>


<style scoped>

</style>
