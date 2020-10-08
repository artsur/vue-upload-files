<template>
    <div :class="root_class">
        <a href="#" :class="btn_class" @click.prevent="addFiles">{{btn_text}}</a>
        <a v-if="!upload_on_select && target && files && files.length"
           href="#" :class="btn_upload_class"
           @click.prevent="uploadFiles">{{btn_upload_text}}</a>
        <div v-if="uploading" class="d-inline-block text-primary mx-3"><span class="spinner-border spinner-border-sm"></span>&nbsp;uploading...</div>
        <div class="d-none">
            <input v-if="multiple" type="file" ref="files" multiple :accept="accept" @change="handleFilesUpload()">
            <input v-else type="file" ref="files" :accept="accept" @change="handleFilesUpload()">
        </div>
        <div v-if="files && files.length && !upload_on_select" class="list-group my-1">
            <div v-for="(file,key) in files" class="file-in-list list-group-item list-group-item-action">
                <div class="row">
                    <div v-if="file.type.match(/image.*/)" class="col-auto">
                        <div class="d-flex h-100 w-100 align-items-center">
                            <img :src="previewImg(file)" class="preview-img">
                        </div>
                    </div>
                    <div class="col">
                        <div class="d-flex h-100 w-100 align-items-center">
                            {{file.name}}
                        </div>
                    </div>
                    <div class="col-auto">
                        <div class="d-flex h-100 w-100 align-items-center">
                            <button class="remove-file ico ico-remove text-danger" @click.prevent="removeFile(key)"></button>
                        </div>
                    </div>
                </div>

            </div>
        </div>
        <div v-if="!upload_on_select && target && files && files.length" class="text-right my-2">
            <a href="#" :class="btn_upload_class" @click.prevent="uploadFiles">{{btn_upload_text}}</a>
        </div>
    </div>
</template>

<script>
    export default {
        name: "UploadFiles",
        props:{
            root_class:{type:String,default:'position-relative'}, //CSS class for component root div
            btn_text:{type:String,default:'Select files'}, //Label for button
            btn_class:{type:String, default:'btn btn-primary'}, //CSS class for button
            accept:{type:String, default:'image/*'}, //File types which will be accepted
            multiple:{type:Boolean, default:false}, //Ability to upload multiple files at once
            target:{type:String,default:null}, //Url for POST method for upload files
            upload_on_select:{type:Boolean, default:false}, //Upload files immediately after selection
            btn_upload_text:{type:String, default:'Upload'}, //Label for upload button
            btn_upload_class:{type:String, default:'btn btn-primary'}, //CSS class for upload button
        },
        data(){
            return{
                files:[],
                uploading:false,
            }
        },
        watch:{
            files(val){
                if(val && val.length) {
                    this.$emit('select',val);
                    if(this.upload_on_select){
                        this.uploadFiles();
                    }
                }
            }
        },
        methods:{
            addFiles(){
                this.$refs.files.click();
            },
            handleFilesUpload(){
                let uploadedFiles = this.$refs.files.files;
                for( let i = 0; i < uploadedFiles.length; i++ ){
                    this.files.push( uploadedFiles[i] );
                }
            },
            removeFile(key){
                this.files.splice(key,1);
            },
            previewImg(file){
                return URL.createObjectURL(file);
            },
            uploadFiles(){
                if(this.target && this.files.length){
                    let formdata = new FormData;
                    for(let i=0; i<this.files.length; i++){
                        let file = this.files[i];
                        formdata.append('upload_files['+i+']',file);
                    }

                    this.uploading = true;
                    axios.post(this.target,
                        formdata,
                        {headers:{'Content-Type': 'multipart/form-data'}}).then(resp=>{
                        if(resp.data){
                            this.files=[];
                            this.$emit('uploaded',resp.data);
                            this.uploading = false;
                        }
                    }).catch(err=>{
                        console.log(err);
                        if(err.response) console.log(err.response);
                        this.uploading = false;
                    });
                }
            }
        }
    }
</script>

<style lang="scss">
    .file-in-list{
        .remove-file{
            border:none;
            background: none;
            outline:none;
            font-size: 1.2rem;
            line-height: 1.2rem;
            padding:0;
            display:none;
        }
        &:hover .remove-file{
            display:inline-block;
        }
        .preview-img{
            width:100px;
        }
    }
</style>
