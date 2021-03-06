<template>
  <div>
    <div v-if="gameState.ready" class="all-items">
      <div class="items-container">
        <RoomItem v-for="item in itemsInRoom"
                  :key="item.id"
                  :item="item"
                  :is-unlocked="isUnlocked(item.id)"
                  @selectImage="selectImage($event)"
                  @toggleLock="adminToggleLock(item.id)"
        />
      </div>
    </div>
    <div v-if="!gameState.ready" class="waiting">
      <p>Gracias por venir</p>
      <p><br /><br /></p>
      <p>Damián llegará a la hora señalada</p>
    </div>
    <SelectedItem v-if="selectedItem"
                  :item="selectedItem"
                  :url="getUrl(selectedItem)"
                  @hide="hideImage()"
    />
  </div>
</template>
<style>
  .all-items, .waiting {
    padding: 5vh 3vw;
    height: 80vh;
    overflow-y: auto;
  }

  .waiting p {
    font-size: 10vh;
    color: #fff;
  }
  .all-items::-webkit-scrollbar {
    background: #b5b5b5;
  }

  .all-items::-webkit-scrollbar-thumb {
    background: #a593a7;
    -webkit-border-radius: 0.2ex;
    -webkit-box-shadow: 0px 1px 2px rgba(77, 58, 18, 0.75);
  }

  .items-container {
    display: grid;
    grid-template-columns: 3.3fr 3.3fr 3.3fr;
    grid-column-gap: 2vw;
    grid-row-gap: 5vh;
  }
</style>
<script>
import SelectedItem from './SelectedItem';
import RoomItem from './RoomItem';
import { db } from '../../../config/db';
import { isAdmin } from '../../../lib/is-admin';
import { isCorruptedForMe } from '../../../lib/is-corrupted-destinatary';

export default {
  name: 'Room',
  components: {
    SelectedItem,
    RoomItem,
  },
  props: {
    activeRoom: {
      type: Number,
      default: 0,
    },
    gameConfig: {
      type: Object,
      default: null,
    }
  },
  data() {
    return {
      publicPath: process.env.BASE_URL,
      gameState: {
        unlockedItems: [],
        ready: false,
      },
      selectedItem: null
    }
  },
  firestore: {
    gameState: db.doc('/game-states/code-nod/'),
  },
  computed: {
    itemsInRoom() {
      return this.gameConfig.items.filter(item => (item.roomId === this.activeRoom));
    }
  },
  methods: {
    adminToggleLock(itemId) {
      if (this.isUnlocked(itemId)) {
        this.gameState.unlockedItems.splice(this.gameState.unlockedItems.indexOf(itemId), 1);
      } else {
        this.gameState.unlockedItems.push(itemId);
      }
      this.$firestoreRefs.gameState.update( { unlockedItems: this.gameState.unlockedItems });
    },
    getUrl(item) {
      if (item.corrupted && !isAdmin() && isCorruptedForMe(item.destinataries)) {
        return `${this.publicPath}game/common/corrupted-image.jpg`;
      }
      return `${this.publicPath}game/${item.roomId}/${item.image}`
    },
    selectImage(item) {
      this.selectedItem = item;
    },
    hideImage() {
      this.selectedItem = null;
    },
    isUnlocked(itemId) {
      return this.gameState.unlockedItems.indexOf(itemId) >= 0;
    }
  }
}
</script>
