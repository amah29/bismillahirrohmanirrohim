<template>
  <div>
    <div class="text-h4 text-center">
      {{ $t('folder.welcome') }}
    </div>

    <FabBtnAddMobile
      @eventClick="dialogoAddPasta = true, resetFolder()"
    />

    <FabBtnAddDesktop
      @eventClick="dialogoAddPasta = true, resetFolder()"
    />

    <div class="q-pa-md divPrincipal">
      <q-list bordered>
        <div class="text-h6 text-center">
          {{ $t('folder.title') }}
        </div>

        <q-item
          v-for="folders in folderList"
          :key="folders.id"
          v-ripple
          clickable
          @click="runBoardScreen(folders)"
        >
          <q-item-section
            avatar
            top
          >
            <q-avatar
              icon="folder"
              color="primary"
              text-color="secondary"
            />
          </q-item-section>

          <q-item-section>
            <q-item-label lines="1">
              {{ folders.title }}
            </q-item-label>
          </q-item-section>

          <q-item-section side>
            <q-icon
              name="edit"
              color="blue"
              @click.stop="showFolderEdit(folders)"
            />
          </q-item-section>

          <q-item-section side>
            <q-icon
              name="delete_sweep"
              color="grey ligten-1"
              @click.stop="showFolderDelete(folders)"
            />
          </q-item-section>
        </q-item>
      </q-list>
    </div>

    <DialogAddFolder
      :message="$t('dialogs.addNewFolder')"
      :label="'Informe o nome da pasta'"
      :dialog="dialogoAddPasta"
      @eventClose="dialogoAddPasta = false"
      @storeEvent="storeFolder($event)"
    />

    <q-dialog v-model="dialogoEditaPasta">
      <q-card
        class="text-center"
        style="width: 500px"
      >
        <q-card-section>
          <div class="text-h6">
            {{ $t('dialogs.editFolder') }}
          </div>
        </q-card-section>

        <q-card-section>
          <q-form class="q-gutter-md">
            <q-input
              v-model="folder.title"
              label="Informe o nome da pasta"
              required
            />
          </q-form>
        </q-card-section>

        <q-card-section align="center">
          <q-btn
            class="q-ma-xs"
            color="black"
            @click.stop="dialogoEditaPasta = false"
          >
            {{ $t('geral.back') }}
          </q-btn>
          <q-btn
            class="q-ma-xs"
            color="green"
            @click="updateFolder"
          >
            {{ $t('geral.yes') }}
          </q-btn>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog v-model="dialogoConfirmaDeletaPasta">
      <q-card>
        <q-card-section>
          <div class="text-h5">
            {{ $t('dialogs.questionDelete') }}
          </div>
        </q-card-section>

        <q-card-section>
          <div class="text-h6">
            {{ folder.title }}
          </div>
        </q-card-section>

        <q-card-section align="center">
          <q-btn
            class="q-ma-xs"
            color="black"
            @click.stop="dialogoConfirmaDeletaPasta = false"
          >
            {{ $t('geral.back') }}
          </q-btn>
          <q-btn
            class="q-ma-xs"
            color="green"
            @click="destroyFolder"
          >
            {{ $t('geral.yes') }}
          </q-btn>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import DialogAddFolder from '../components/dialogs/DialogAddFolderBoard'
import FabBtnAddDesktop from '../components/button/FabAddDesktop'
import FabBtnAddMobile from '../components/button/FabAddMobile'
export default {
  name: 'Dash',
  components: {
    DialogAddFolder,
    FabBtnAddDesktop,
    FabBtnAddMobile
  },
  data () {
    return {
      dialogoAddPasta: false,
      dialogoEditaPasta: false,
      dialogoConfirmaDeletaPasta: false,
      folder: {
        id: null,
        title: '',
        createdAt: null,
        updatedAt: null
      },
      ref: this.$firebase.firestore().collection('app'),
      folderList: []
    }
  },
  computed: {
    refFolder () {
      if (this.user.uid != null) {
        return this.$firebase
          .firestore()
          .collection('app')
          .doc(this.user.uid)
          .collection('folder')
      } else {
        return null
      }
    }
  },
  watch: {
    refFolder: 'indexFolder'
  },
  mounted () {
    this.$store.dispatch('setCurrentFolder', { id: null })
    this.$store.dispatch('setCurrentBoard', { id: null })
    this.indexFolder()
  },

  methods: {
    storeFolder (title) {
      this.folder.title = title
      if (this.folder.title.includes('/') || this.folder.title.includes('..')) {
        return
      }
      const newFolder = {
        title: this.folder.title,
        id: null,
        createdAt: this.$timestamp,
        updatedAt: this.$timestamp
      }

      this.$db.collection('app')
        .doc(this.user.uid)
        .collection('folder')
        .add(newFolder)
        .then(ref => {
          const pushID = { id: ref.id }
          ref.update(pushID)
          this.$notifiy(this.$t('alert.sucess.addedFolder'), 'green')
        })
        .catch(() => {
          this.$notifiy(this.$t('alert.error.errorTryingToAdd'), 'red')
        })

      this.dialogoAddPasta = false
      this.resetFolder()
    },
    indexFolder () {
      if ((this.user.uid != null) & (this.refFolder != null)) {
        this.refFolder.onSnapshot(querySnapshot => {
          this.folderList = []
          querySnapshot.forEach(doc => {
            this.folderList.push(doc.data())
          })
        })
      }
    },
    updateFolder () {
      if (this.folder.title.includes('/') || this.folder.title.includes('..')) {
        return
      }
      const newFolder = {
        title: this.folder.title,
        updatedAt: this.$timestamp
      }
      this.$db.collection('app')
        .doc(this.user.uid)
        .collection('folder')
        .doc(this.folder.id)
        .update(newFolder)
        .then(() => {
          this.$notifiy(this.$t('alert.sucess.updatedFolder'), 'green')
        })
        .catch(() => {
          this.$notifiy(this.$t('alert.error.errorTryingToUpdated'), 'red')
        })
      this.dialogoEditaPasta = false
      this.resetFolder()
    },
    destroyFolder () {
      this.$db.collection('app')
        .doc(this.user.uid)
        .collection('folder')
        .doc(this.folder.id)
        .delete()
        .then(() => {
          this.$notifiy(this.$t('alert.sucess.removedFolder'), 'green')
        })
        .catch(() => {
          this.$notifiy(this.$t('alert.error.errorTryingToRemove'), 'red')
        })
      this.dialogoConfirmaDeletaPasta = false
    },
    runBoardScreen (folder) {
      this.$router.push({
        name: 'Board',
        params: { idFolder: (folder.id).toString() }
      })
    },
    showFolderEdit (folder) {
      this.dialogoEditaPasta = true
      this.folder = folder
    },
    showFolderDelete (folder) {
      this.dialogoConfirmaDeletaPasta = true
      this.folder = folder
    },
    resetFolder () {
      this.folder = {
        id: null,
        title: '',
        createdAt: null,
        updatedAt: null
      }
    }
  }
}
</script>

<style scoped>
form > * {
  display: block;
}
</style>
