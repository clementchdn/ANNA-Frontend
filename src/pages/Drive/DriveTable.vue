<template>
  <div>
    <span
      style="font-size: 1.1em; cursor: pointer; user-select:none;"
      @click="goHome"
    >
      <i class="fas fa-home" /> <span style="text-decoration: underline"> root</span> <span v-if="folder.name !== 'root'"> > </span>
    </span>
    <span v-if="folder.name !== 'root' && folder.dirTree.length <= 2">
      <span
        v-for="parent in folder.dirTree"
        :key="parent.id"
        style="font-size: 1.1em; cursor: pointer; user-select:none;"
        @click="openFolder(parent)"
      >
        <span style="text-decoration: underline">{{ wrapName(parent.name) }}</span> <span> > </span>
      </span>
    </span>
    <span
      v-if="folder.dirTree.length > 2"
      style="cursor: default;"
    > ... > </span>
    <span v-if="folder.name !== 'root' && folder.dirTree.length > 2">
      <span
        v-for="parent in altDirTree"
      
        :key="parent.id"
        style="font-size: 1.1em; cursor: pointer; user-select:none;"
        :title="parent.name"
        @click="openFolder(parent)"
      >
        <span style="text-decoration: underline">{{ wrapName(parent.name) }}</span> <span> {{ parent.arrow }} </span>
      </span>
    </span>

    <table
      v-if="showHistory"
      id="file-history"
      style="margin: 0.3em 0; user-select: none"
    >
      <tr
        class="pas-toi no-hover"
        style="cursor: default"
      >
        <th>Type<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Name <!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--> </th>
        <th>Serial nbr<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Owner <!--i @click='clique2' class="fas fa-caret-up" :class='classR2'></i--> </th>
        <th v-if="showHistory">
          Date
        </th>
        <th>Size <!--i @click='clique3' class="fas fa-caret-up" :class='classR3'></i--> </th>
      </tr>
      <tr>
        <th colspan="6">
          <p
            class="center"
            style="color: #7a7a7a"
          >
            File History
          </p>
        </th>
      </tr>
      <tr
        v-for="rev in existingRevs"
        :key="rev.id"
        :class="{selected: rev.id === selectedFile.id}"
        @click="select(rev)"
        @dblclick="openFolder(rev)"
      >
        <td v-html="getIcon(rev)" />
        <td :title="rev.name">
          {{ wrapName(rev.name) }}
        </td>
        <td>
          {{ wrapName2(rev.serialNbr) }}
        </td>
        <td>
          {{ wrapName($store.getters.users.find(user => user.id == rev.ownerId).username) }}
        </td>
        <td>
          {{ getDate(rev.updatedAt) }}
        </td>
        <td v-if="rev.size > 0">
          {{ convertSize(rev) }}
        </td>
        <td v-else />
      </tr>
    </table>
    <table
      v-if="keyword && keyword.trim().length >= 2"
      id="result-search"
      style="margin: 0.3em 0; user-select: none"
    >
      <tr
        class="pas-toi no-hover"
        style="cursor: default"
      >
        <th>Type<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Name <!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--> </th>
        <th>Serial nbr<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Owner <!--i @click='clique2' class="fas fa-caret-up" :class='classR2'></i--> </th>
        <th v-if="showHistory">
          Date
        </th>
        <th>Size <!--i @click='clique3' class="fas fa-caret-up" :class='classR3'></i--> </th>
      </tr>
      <tr
        v-for="file in results"
        :key="file.fileId"
        :class="{selected: file.fileId === selectedFile.fileId}"
        @click="select(file)"
        @dblclick="openFolder(file)"
      >
        <td v-html="getIcon(file)" />
        <td :title="file.name">
          {{ wrapName(file.name) }}
        </td>
        <td>
          {{ wrapName2(file.serialNbr) }}
        </td>
        <td v-if="file.owner">
          {{ wrapName(file.owner.username) }}
        </td>
        <td v-if="file.size > 0">
          {{ convertSize(file) }}
        </td>
        <td v-else />
      </tr>
      <tr v-if="results.length === 0">
        <p class="center">
          No Results
        </p>
      </tr>
    </table>
    <table
      v-if="!(keyword && keyword.trim().length >= 2) && !showHistory"
      id="inside-folder-list"
      style="margin: 0.3em 0; user-select: none"
    >
      <tr
        class="pas-toi no-hover"
        style="cursor: default"
      >
        <th>Type<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Name <!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--> </th>
        <th>Serial nbr<!--i @click="clique" class="fas fa-caret-up" :class='classR'></i--></th>
        <th>Owner <!--i @click='clique2' class="fas fa-caret-up" :class='classR2'></i--> </th>
        <th v-if="showHistory">
          Date
        </th>
        <th>Size <!--i @click='clique3' class="fas fa-caret-up" :class='classR3'></i--> </th>
      </tr>
      <tr
        v-for="file in content"
        :key="file.fileId"
        :class="{selected: file.fileId === selectedFile.fileId}"
        @click="select(file)"
        @dblclick="openFolder(file)"
      >
        <td v-html="getIcon(file)" />
        <td :title="file.name">
          {{ wrapName(file.name) }}
        </td>
        <td>
          {{ wrapName2(file.serialNbr) }}
        </td>
        <td>
          {{ wrapName(file.owner.username) }}
        </td>
        <td v-if="file.size > 0">
          {{ convertSize(file) }}
        </td>
        <td v-else />
      </tr>
    </table>
    <a
      v-if="content.length === 0"
      style="text-align: center; font-size: 20px; width: 100%;"
      @click.prevent="$modal.show('uploadFile', {isFolder: false, isEditing: false})"
    >
      This folder is still empty.
    </a>
  </div>
</template>

<script>
import store from '@/modules/store';
import FileSize from 'filesize';
import moment from 'moment';

export default {
    data() {
        return {
            loading: false,
            classR: '',
            classR2: '',
            classR3: '',
            altDirTree: []
        };
    },
    computed: {
        results() {
            return store.getters.searchResultsContent;
        },
        keyword() {
            return store.getters.keyWord;
        },
        folder() {
            return store.getters.folder;
        },
        content() {
            return store.getters.content;
        },
        selectedFile() {
            return store.getters.selectedFile;
        },
        showHistory() {
            return store.getters.showHistory;
        },
        metaData() {
            return store.getters.metaData;
        },
        foldersList() {
            return store.getters.foldersList;
        },
        existingRevs() {
            return store.getters.metaData.filter(x => x.exists);
        }
    },
    mounted() {
        if (this.folder && this.folder.dirTree.length > 2) {
            let a = Array();
            a[0] = {
                fileId: this.folder.dirTree[this.folder.dirTree.length -2].fileId,
                name: this.folder.dirTree[this.folder.dirTree.length -2].name,
                arrow: '>',
                type: 'folder'
            };
            a[1] = {
                fileId: this.folder.dirTree[this.folder.dirTree.length -1].fileId,
                name: this.folder.dirTree[this.folder.dirTree.length -1].name,
                arrow: '',
                type: 'folder'
            };
            this.altDirTree = a;
        }
    },
    methods: {
        addClass(e) {
            e.addClass('rotate');
        },
        clique(){
            if (this.classR === 'rotate') {
                this.classR = '';
            }else if (this.classR === '') {
                this.classR = 'rotate';
            }
        },
        clique2(){
            if (this.classR2 === 'rotate') {
                this.classR2 = '';
            }else if (this.classR2 === '') {
                this.classR2 = 'rotate';
            }
        },
        clique3(){
            if (this.classR3 === 'rotate') {
                this.classR3 = '';
            }else if (this.classR3 === '') {
                this.classR3 = 'rotate';
            }
        },
        select(file) {
            if (file.fileId === this.selectedFile.fileId && this.selectedFile.exists==undefined) store.dispatch('selectFile', {});
            else store.dispatch('selectFile', file);
        },
        wrapName(name) {
            if (name && name.length > 38)
                return name.substring(0, 38) + '...';
            else
                return name;
        },
        wrapName2(name) {
            if (name && name.length > 16)
                return name.substring(0, 16) + '...';
            else
                return name;
        },
        getDate(date) {
            return moment(date).format('YYYY-MM-DD');
        },
        getIcon(file) {
            if (file.type === 'folder') {
                return '<i class="fa fa-folder" aria-hidden="true"></i>';
            }

            if(!file.type) { file.type = ''; }
            switch (file.type) {

                // PDF
                case 'application/pdf':
                    return '<i class="fa fa-file-pdf" aria-hidden="true"></i>';

                    // Archive
                case 'application/zip':
                    return '<i class="fa fa-file-archive" aria-hidden="true"></i>';

                    // Image
                case 'image/gif':
                    return '<i class="fas fa-file-image"></i>';
                case 'image/png':
                    return '<i class="fas fa-file-image"></i>';
                case 'image/jpeg':
                    return '<i class="fas fa-file-image"></i>';
                case 'image/tiff':
                    return '<i class="fa fa-file-image" aria-hidden="true"></i>';

                    // Audio
                case 'audio/mpeg':
                    return '<i class="fas fa-file-audio"></i>';
                case 'audio/x-ms-wma':
                    return '<i class="fas fa-file-audio"></i>';
                case 'audio/vnd.rn-realaudio':
                    return '<i class="fas fa-file-audio"></i>';
                case 'audio/x-wav':
                    return '<i class="fas fa-file-audio"></i>';

                    // Video
                case 'video/mpeg':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/mp4':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/quicktime':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/x-ms-wmv':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/x-msvideo':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/x-flv':
                    return '<i class="fas fa-file-video"></i>';
                case 'video/webm':
                    return '<i class="fas fa-file-video"></i>';

                case 'folder':
                    return '<i class="fa fa-file-folder" aria-hidden="true"></i>';

                case '':
                default:
                    return '<i class="fa fa-file" aria-hidden="true"></i>';
            }
        },
        convertSize(file) {
            if(file.type === 'folder') {
                return '';
            }
            return FileSize(file.size);
        },
        async openFolder(file) {
            if (file.type === 'folder' || !file.type) {
                this.loading = true;
                await store.dispatch('retrieveFolder', file.fileId)
                    .then(() => store.dispatch('selectFile', {}))
                    .then(() => this.loading = false);
                if (this.folder.dirTree.length > 2) {
                    let a = Array();
                    a[0] = {
                        fileId: this.folder.dirTree[this.folder.dirTree.length -2].fileId,
                        name: this.folder.dirTree[this.folder.dirTree.length -2].name,
                        arrow: '>',
                        type: 'folder'
                    };
                    a[1] = {
                        fileId: this.folder.dirTree[this.folder.dirTree.length -1].fileId,
                        name: this.folder.dirTree[this.folder.dirTree.length -1].name,
                        arrow: '',
                        type: 'folder'
                    };
                    this.altDirTree = a;
                }
            }
        },
        async goHome() {
            this.loading = true;
            await store.dispatch('retrieveFolder', 1)
                .then(() => store.dispatch('selectFile', {}))
                .then(() => this.loading = false);
        }
    }
};
</script>
