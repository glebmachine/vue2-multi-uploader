<template>
    <div class="uploadBox">
        <form role="form" enctype="multipart/form-data" @submit.prevent="onSubmit">
            <div class="uploadBoxMain" v-if="!itemsAdded">
                <div class="form-group">
                    <div class="dropArea" @ondragover="onChange">
                        <div class="drop-area__description" style="font-size: 18px; padding-bottom: 70px;"><span>Выберите фото</span> или перетащите сюда</div>
                        <input type="file" id="items" name="items[]" required multiple @change="onChange">
                        <!-- <p class="help-block">Space for your instructions</p> -->
                    </div>
                </div>
            </div>
            <div class="uploadBoxMain" v-else>
<!--                 <p><strong>Names</strong></p> -->
                <!-- <ol> -->
                <ul>
                    <li style="list-style:none" v-for="name in itemsNames">{{name}}</li>
                </ul>
                <!-- </ol> -->
<!--
                <p><strong>Sizes</strong></p>
                <ol>
                    <li v-for="size in itemsSizes">{{size}}</li>
                </ol>
                <p><strong>Total files:</strong> {{itemsAdded}}</p>
                <p><strong>Total upload size:</strong> {{itemsTotalSize}}</p>
-->
                <button @click="removeItems" class="button remove-files component-report-new__submit">Удалить файл</button>
                <!-- Loader -->
                <div class="loader" v-if="isLoaderVisible">
                    <div class="loaderImg"></div>
                </div>
                <!-- End Loader -->
            </div>
<!--
            <div>
                <button type="submit" class="btn btn-primary btn-black btn-round" :disabled="itemsAdded < minItems || itemsAdded > maxItems">Upload</button>
                <button type="button" class="btn btn-default btn-round" @click="removeItems">Cancel</button>
            </div>
-->
            <div class="successMsg" v-if="successMsg !== ''">{{successMsg}}</div>
            <div class="errorMsg" v-if="errorMsg !== ''">An error has occurred:<br>{{errorMsg}}</div>
            <div class="errorMsg" v-if="itemsAdded && itemsAdded < minItems">Minimum {{minItems}} files need to be added to uploader. Please remove the files and try again.</div>
            <div class="errorMsg" v-if="itemsAdded && itemsAdded > maxItems">A maximum of {{maxItems}} files can be uploaded each time. Please remove the files and try again.</div>
        </form>
    </div>
</template>

<script>
require('es6-promise').polyfill();
import axios from 'axios';

component: {axios}
export default {
    props: {
        postURL: {
            type: String,
            required: true
        },
        minItems: {
            type: Number,
            default: 1
        },
        maxItems: {
            type: Number,
            default: 30
        },
        method: {
            type: String,
            default: 'post'
        },
        postMeta: {
            type: [String, Array, Object],
            default: ''
        },
        successMessagePath: {
            type: String,
            required: true
        },
        errorMessagePath: {
            type: String,
            required: true
        }
    },

    /*
     * The component's data.
     */
    data() {
        return {
            items: [],
            itemsAdded: '',
            itemsNames: [],
            itemsSizes: [],
            itemsTotalSize: '',
            formData: '',
            successMsg: '',
            errorMsg: '',
            isLoaderVisible: false,
        }
    },

    methods: {
        // http://scratch99.com/web-development/javascript/convert-bytes-to-mb-kb/
        bytesToSize(bytes) {
            const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
            if (bytes === 0) return 'n/a';
            let i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)));
            if (i === 0) return bytes + ' ' + sizes[i];
            return (bytes / Math.pow(1024, i)).toFixed(2) + ' ' + sizes[i];
        },

        onChange(e) {
            this.successMsg = '';
            this.errorMsg = '';
            this.formData = new FormData();
            let files = e.target.files || e.dataTransfer.files;
            this.itemsAdded = files.length;
            let fileSizes = 0;
            console.log(files)
            this.$store.state.loadFiles = files;
            for (let x in files) {
                if (!isNaN(x)) {
                    this.items = e.target.files[x] || e.dataTransfer.files[x];
                    this.itemsNames[x] = files[x].name;
                    this.itemsSizes[x] = this.bytesToSize(files[x].size);
                    fileSizes += files[x].size;
                    this.formData.append('items[]', this.items);
                }
            }
            this.itemsTotalSize = this.bytesToSize(fileSizes);
        },

        removeItems() {
            this.items = '';
            this.itemsAdded = '';
            this.itemsNames = [];
            this.itemsSizes = [];
            this.itemsTotalSize = '';
        },

        onSubmit() {
            this.isLoaderVisible = true;

            if ((typeof this.postMeta === 'string' && this.postMeta !== '') ||
                (typeof this.postMeta === 'object' && Object.keys(this.postMeta).length > 0)) {
                this.formData.append('postMeta', this.postMeta);
            }

            if (this.method === 'put' || this.method === 'post' ) {
                axios({method: this.method, url: this.postURL, data: this.formData})
                    .then((response) => {
                        this.isLoaderVisible = false;
                        // Show success message
                        this.successMsg = response + "." + this.successMessagePath;
                        this.removeItems();
                    })
                    .catch((error) => {
                        this.isLoaderVisible = false;
                        this.errorMsg = error + "." + this.errorMessagePath;
                        this.removeItems();
                    });
            } else {
                this.errorMsg = "This HTTP method is not allowed. Please use either 'put' or 'post' methods.";
                this.removeItems();
            }
        },
    }
}
</script>

<style>
.uploadBox {
    position: relative;
    border: 3px dashed #7a7979;
    min-height: 171px;

    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
}

.uploadBox h3 {
    padding-top: 1em;
}

.uploadBox .uploadBoxMain {
    position: relative;
}

/* Drag and drop */
.uploadBox .dropArea {
    position: relative;
    width: 100%;
    text-align: center;
    font-size: 2em;
    padding-top: 80px;
}

.uploadBox .dropArea input {
    position: absolute;
    cursor: pointer;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
}
/* End drag and drop */

/* Loader */
.uploadBox .loader {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    opacity: 0.9;
}

.uploadBox .loaderImg {
    border: 9px solid #eee;
    border-top: 9px solid #00ADCE;
    border-radius: 50%;
    width: 70px;
    height: 70px;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}
/* End Loader */

.uploadBox .errorMsg {
    font-size: 2em;
    color: #a94442;
}

.uploadBox .successMsg {
    font-size: 2em;
    color: #3c763d;
}

.remove-files{
    width: 190px;
    height: 30px;
    text-align: center;
    justify-content: center;
    margin: 12px auto 0;
    padding: 0;
}
</style>
