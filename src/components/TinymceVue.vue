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
                    return import('tinymce/tinymce')
                        .then(() => {
                            const promises = [];

                            promises.push(import('tinymce/themes/modern/theme'));

                            promises.push(import('tinymce/plugins/advlist'));
                            promises.push(import('tinymce/plugins/wordcount'));
                            promises.push(import('tinymce/plugins/autolink'));
                            promises.push(import('tinymce/plugins/autosave'));
                            promises.push(import('tinymce/plugins/charmap'));
                            promises.push(import('tinymce/plugins/codesample'));
                            promises.push(import('tinymce/plugins/contextmenu'));
                            promises.push(import('tinymce/plugins/emoticons'));
                            promises.push(import('tinymce/plugins/fullscreen'));
                            promises.push(import('tinymce/plugins/hr'));
                            promises.push(import('tinymce/plugins/imagetools'));
                            promises.push(import('tinymce/plugins/insertdatetime'));
                            promises.push(import('tinymce/plugins/link'));
                            promises.push(import('tinymce/plugins/media'));
                            promises.push(import('tinymce/plugins/noneditable'));
                            promises.push(import('tinymce/plugins/paste'));
                            promises.push(import('tinymce/plugins/print'));
                            promises.push(import('tinymce/plugins/searchreplace'));
                            promises.push(import('tinymce/plugins/tabfocus'));
                            promises.push(import('tinymce/plugins/template'));
                            promises.push(import('tinymce/plugins/textpattern'));
                            promises.push(import('tinymce/plugins/visualblocks'));
                            promises.push(import('tinymce/plugins/anchor'));
                            promises.push(import('tinymce/plugins/autoresize'));
                            promises.push(import('tinymce/plugins/bbcode'));
                            promises.push(import('tinymce/plugins/code'));
                            promises.push(import('tinymce/plugins/colorpicker'));
                            promises.push(import('tinymce/plugins/directionality'));
                            promises.push(import('tinymce/plugins/fullpage'));
                            promises.push(import('tinymce/plugins/help'));
                            promises.push(import('tinymce/plugins/image'));
                            promises.push(import('tinymce/plugins/importcss'));
                            promises.push(import('tinymce/plugins/legacyoutput'));
                            promises.push(import('tinymce/plugins/lists'));
                            promises.push(import('tinymce/plugins/nonbreaking'));
                            promises.push(import('tinymce/plugins/pagebreak'));
                            promises.push(import('tinymce/plugins/preview'));
                            promises.push(import('tinymce/plugins/save'));
                            promises.push(import('tinymce/plugins/spellchecker'));
                            promises.push(import('tinymce/plugins/table'));
                            promises.push(import('tinymce/plugins/textcolor'));
                            promises.push(import('tinymce/plugins/toc'));
                            promises.push(import('tinymce/plugins/visualchars'));
                            
                            promises.push(import('tinymce/skins/lightgray/skin.min.css'));

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
