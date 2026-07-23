<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons v-if="currentPathParts.length > 0" slot="start">
          <ion-button aria-label="Zurück" @click="navigateBack">
            <ion-icon slot="icon-only" :icon="arrowBackOutline" />
          </ion-button>
        </ion-buttons>
        <ion-title>{{ pageTitle }}</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">{{ pageTitle }}</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-list>
        <ion-item
          v-for="entry in directoryEntries"
          :key="entry.name"
          button
          @click="entry.type === 'directory' ? openDirectory(entry) : openFile(entry)"
        >
          <ion-icon
            slot="start"
            :icon="entry.type === 'directory' ? folderOutline : documentOutline"
          />
          <ion-label>{{ entry.name }}</ion-label>
          <ion-button
            slot="end"
            fill="clear"
            aria-label="Löschen"
            @click.stop="requestDelete(entry)"
          >
            <ion-icon slot="icon-only" :icon="trashOutline" />
          </ion-button>
        </ion-item>
      </ion-list>

      <ion-fab slot="fixed" vertical="bottom" horizontal="end">
        <ion-fab-button @click="isAddActionSheetOpen = true">
          <ion-icon :icon="addOutline" />
        </ion-fab-button>
      </ion-fab>
    </ion-content>

    <ion-action-sheet
      :is-open="isAddActionSheetOpen"
      :buttons="addActionSheetButtons"
      @didDismiss="isAddActionSheetOpen = false"
    />

    <ion-alert
      :is-open="isCreateFolderAlertOpen"
      header="Ordner erstellen"
      :inputs="createFolderAlertInputs"
      :buttons="createFolderAlertButtons"
      @didDismiss="isCreateFolderAlertOpen = false"
    />

    <ion-alert
      :is-open="isDeleteAlertOpen"
      :header="deleteAlertQuestion"
      :buttons="deleteAlertButtons"
      @didDismiss="closeDeleteAlert"
    />
  </ion-page>
</template>

<script setup lang="ts">
import { FileOpener } from '@capawesome-team/capacitor-file-opener';
import { FilePicker } from '@capawesome/capacitor-file-picker';
import { Directory, Filesystem, type FileInfo } from '@capacitor/filesystem';
import {
  IonActionSheet,
  IonAlert,
  IonButton,
  IonButtons,
  IonContent,
  IonFab,
  IonFabButton,
  IonHeader,
  IonIcon,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
  onIonViewWillEnter,
} from '@ionic/vue';
import {
  addOutline,
  arrowBackOutline,
  documentOutline,
  folderOutline,
  trashOutline,
} from 'ionicons/icons';
import { computed, ref } from 'vue';

const managedFilesRoot = 'dateimanager-files';
const currentPathParts = ref<string[]>([]);
const directoryEntries = ref<FileInfo[]>([]);
const isAddActionSheetOpen = ref(false);
const isCreateFolderAlertOpen = ref(false);
const isDeleteAlertOpen = ref(false);
const entryToDelete = ref<FileInfo | null>(null);

const addActionSheetButtons = [
  { text: 'Datei hinzufügen', handler: () => addFile() },
  {
    text: 'Ordner erstellen',
    handler: () => {
      isCreateFolderAlertOpen.value = true;
    },
  },
  { text: 'Abbrechen', role: 'cancel' },
];

const createFolderAlertInputs = [
  {
    name: 'folderName',
    type: 'text' as const,
    placeholder: 'Ordnername',
  },
];

const currentPath = computed(() =>
  [managedFilesRoot, ...currentPathParts.value].join('/'),
);
const pageTitle = computed(
  () => currentPathParts.value[currentPathParts.value.length - 1] ?? 'Dateimanager',
);
const deleteAlertQuestion = computed(() =>
  entryToDelete.value
    ? `Soll "${entryToDelete.value.name}" gelöscht werden?`
    : 'Soll der ausgewählte Eintrag gelöscht werden?',
);

const ensureManagedFilesRoot = async () => {
  let rootExists = false;

  try {
    const rootInfo = await Filesystem.stat({
      directory: Directory.Data,
      path: managedFilesRoot,
    });
    rootExists = rootInfo.type === 'directory';
  } catch {
    rootExists = false;
  }

  if (!rootExists) {
    await Filesystem.mkdir({
      directory: Directory.Data,
      path: managedFilesRoot,
      recursive: true,
    });
  }
};

const loadDirectoryEntries = async () => {
  const result = await Filesystem.readdir({
    directory: Directory.Data,
    path: currentPath.value,
  });

  directoryEntries.value = result.files;
};

const openDirectory = async (entry: FileInfo) => {
  if (entry.type !== 'directory') {
    return;
  }

  currentPathParts.value.push(entry.name);
  await loadDirectoryEntries();
};

const openFile = async (entry: FileInfo) => {
  if (entry.type !== 'file') {
    return;
  }

  const filePath = [managedFilesRoot, ...currentPathParts.value, entry.name].join('/');
  const { uri } = await Filesystem.getUri({
    directory: Directory.Data,
    path: filePath,
  });

  await FileOpener.openFile({ path: uri });
};

const navigateBack = async () => {
  if (currentPathParts.value.length === 0) {
    return;
  }

  currentPathParts.value.pop();
  await loadDirectoryEntries();
};

const requestDelete = (entry: FileInfo) => {
  entryToDelete.value = entry;
  isDeleteAlertOpen.value = true;
};

const closeDeleteAlert = () => {
  isDeleteAlertOpen.value = false;
  entryToDelete.value = null;
};

const deleteSelectedEntry = async () => {
  const entry = entryToDelete.value;

  if (!entry) {
    return false;
  }

  const entryPath = [managedFilesRoot, ...currentPathParts.value, entry.name].join('/');

  if (entry.type === 'file') {
    await Filesystem.deleteFile({
      directory: Directory.Data,
      path: entryPath,
    });
  } else {
    await Filesystem.rmdir({
      directory: Directory.Data,
      path: entryPath,
      recursive: true,
    });
  }

  await loadDirectoryEntries();
};

const addFile = async () => {
  try {
    const result = await FilePicker.pickFiles({
      readData: true,
      limit: 1,
    });
    const file = result.files[0];

    if (!file || file.data === undefined) {
      return;
    }

    const filePath = [managedFilesRoot, ...currentPathParts.value, file.name].join('/');

    await Filesystem.writeFile({
      directory: Directory.Data,
      path: filePath,
      data: file.data,
    });
    await loadDirectoryEntries();
  } catch (error) {
    if (error instanceof Error && error.message === 'pickFiles canceled.') {
      return;
    }

    throw error;
  }
};

const createFolder = async (values: { folderName?: string }) => {
  const folderName = values.folderName?.trim() ?? '';

  if (!folderName) {
    return false;
  }

  const folderPath = [managedFilesRoot, ...currentPathParts.value, folderName].join('/');

  await Filesystem.mkdir({
    directory: Directory.Data,
    path: folderPath,
  });
  await loadDirectoryEntries();
};

const createFolderAlertButtons = [
  { text: 'Abbrechen', role: 'cancel' },
  { text: 'Erstellen', handler: createFolder },
];

const deleteAlertButtons = [
  { text: 'Abbrechen', role: 'cancel' },
  { text: 'Löschen', handler: deleteSelectedEntry },
];

onIonViewWillEnter(async () => {
  await ensureManagedFilesRoot();
  await loadDirectoryEntries();
});
</script>
