<template>
  <div class="home-page mx-auto d-flex justify-center align-center">
    <div style="width: 100%" class="d-flex flex-column">
      <div class="d-flex justify-center mb-6">
        <img height="50" src="images/DoneDoneLogo.svg" alt="Done Done Logo" />
      </div>
      <v-text-field
        v-model.trim="topic"
        outlined
        label="Type a topic for your board..."
        append-icon="mdi-plus-circle"
        :hint="hint"
        @keydown.enter="createBoard"
        @click:append="createBoard"
      ></v-text-field>
      <div v-if="boards.length > 0">
        <h6>Recent Boards</h6>
        <v-chip-group column>
          <v-chip
            v-for="board in boards"
            :key="board"
            :to="`/${board}`"
            close
            @click:close="onBoardRemove(board)"
          >
            {{ board }}
          </v-chip>
        </v-chip-group>
      </div>
    </div>

    <v-dialog v-model="dialog" persistent max-width="290">
      <v-card>
        <v-card-title class="headline red--text">
          Deleting board...
        </v-card-title>
        <v-card-text>
          Are you sure you want to delete board
          <strong>{{ selectedBoardId }}</strong>
          ? This action cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="dialog = false">
            Cancel
          </v-btn>
          <v-btn color="red darken-1" dark @click="removeBoard">
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { customAlphabet } from 'nanoid'
import kebabCase from 'lodash.kebabcase'
import { db } from '@/settings/db'

export default {
  name: 'Home',
  data() {
    return {
      topic: '',
      uniqueId: '',
      boards: [],
      dialog: false,
      selectedBoardId: ''
    }
  },
  created() {
    const nanoid = customAlphabet('1234567890abcdefghijklmnopqrstuvwxyz', 10)
    this.uniqueId = nanoid()

    const jsonBoardList = localStorage.getItem('boards')
    if (jsonBoardList) {
      this.boards = JSON.parse(jsonBoardList)
    }
  },
  computed: {
    hint() {
      return this.topic ? `${kebabCase(this.topic)}-${this.uniqueId}` : ''
    }
  },
  methods: {
    async createBoard() {
      if (this.topic) {
        await db
          .collection('boards')
          .doc(this.hint)
          .set({
            boardName: this.hint
          })
        await db
          .collection('boards')
          .doc(this.hint)
          .collection('list')
          .add({
            taskTitle: 'Welcome to your board!',
            status: 'pending',
            order: 1,
            type: 'task'
          })
        const jsonBoardList = localStorage.getItem('boards')
        if (jsonBoardList) {
          const boardList = JSON.parse(jsonBoardList)
          boardList.push(this.hint)
          localStorage.setItem('boards', JSON.stringify(boardList))
        } else {
          localStorage.setItem('boards', JSON.stringify([this.hint]))
        }
        this.$router.push(`/${this.hint}`)
      }
    },
    onBoardRemove(board) {
      this.dialog = true
      this.selectedBoardId = board
    },
    async removeBoard() {
      await db
        .collection('boards')
        .doc(this.selectedBoardId)
        .delete()
      this.boards = this.boards.filter(board => board !== this.selectedBoardId)
      localStorage.setItem('boards', JSON.stringify(this.boards))
      this.dialog = false
    }
  }
}
</script>

<style scoped>
.home-page {
  max-width: 720px;
  height: 100vh;
}

@media (max-width: 640px) {
  .home-page {
    padding: 0 1rem;
  }
}
</style>