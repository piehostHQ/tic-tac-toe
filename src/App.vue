<script setup>
import { ref, computed, onMounted } from 'vue'
import PieSocket from "piesocket-js";

let player = ref('X')
let board = ref([
    ['', '', ''],
    ['', '', ''],
    ['', '', '']
])
let self = ref('');
let opp = ref('');
let channel;

let userId = ref(Date.now().toString());
const handleSubscribe = () => {
    var pieSocket = new PieSocket({
		clusterId: "YOUR CLUSTER ID",
		apiKey: "YOUR API KEY",
        notifySelf: true,
        presence: true,
        userId: userId.value
    });

    pieSocket.subscribe("chat-room").then(ch => {
        channel = ch;
        console.log("Channel is ready")

        channel.listen("system:member_joined", function (data) {
            console.log("User joined")
            if (data.member.user == userId.value) {
                self.value = data.member
                player.value = 'X'
            }
            else {
                opp.value = data.member;
                player.value = 'O'
            }
        })

        channel.listen('play-turn', function (data) {
            if (data.player != player.value) {
                board.value[data.x][data.y] = data.player;
                player.value = player.value === 'X' ? 'O' : 'X'
            }
        })


        channel.listen('play-turn', function (data) {
            if (data.player != player.value) {
                board.value[data.x][data.y] = data.player;
            }
        })
    })
}

onMounted(handleSubscribe);

const CalculateWinner = (board) => {
    const lines = [[0, 1, 2], [3, 4, 5], [6, 7, 8], [0, 3, 6], [1, 4, 7], [2, 5, 8], [0, 4, 8], [2, 4, 6]]
    for (let i = 0; i < lines.length; i++) {
        const [a, b, c] = lines[i]
        if (board[a] && board[a] === board[b] && board[a] === board[c]) {
            return board[a]
        }
    }
    return null
}

const winner = computed(() => CalculateWinner(board.value.flat()))

const MakeMove = (x, y) => {
    if (winner.value) return
    if (board.value[x][y]) return
    board.value[x][y] = player.value
    channel.publish('play-turn', { x, y, player: player.value });
}
const isTie = computed(() => {
    return board.value.flat().every(cell => cell !== '') && !winner.value;
});

const ResetGame = () => {
    board.value = [
        ['', '', ''],
        ['', '', ''],
        ['', '', '']
    ]
    player.value = 'X'
}
</script>

<template>
    <main class="pt-8 text-center">
        <h1 class="mb-8 text-6xl font-bold uppercase">Tic Tac Toe</h1>

        <h3 class="text-xl mb-4">Player {{ player }}'s turn</h3>

        <div class="flex flex-col items-center mb-8">
            <div v-for="(row, x) in board" :key="x" class="flex">
                <div v-for="(cell, y) in row" :key="y" @click="MakeMove(x, y)"
                    :class="`border border-white w-24 h-24 hover:bg-gray-700 flex items-center justify-center material-icons-outlined text-4xl cursor-pointer ${cell === 'X' ? 'text-pink-500' : 'text-blue-400'}`">
                    {{ cell === 'X' ? 'close' : cell === 'O' ? 'circle' : '' }}
                </div>
            </div>
        </div>

        <div class="text-center">
            <h2 v-if="winner" class="text-6xl font-bold mb-8">Player '{{ winner }}' wins!</h2>
            <h2 v-else-if="isTie" class="text-6xl font-bold mb-8">It's a tie!</h2>
            <button v-if="winner || isTie" @click="ResetGame"
                class="px-4 py-2 bg-pink-500 rounded uppercase font-bold hover:bg-pink-600 duration-300">Reset</button><br>
        </div>
    </main>
</template>

<style>
body {
    @apply bg-gray-800 text-white;
}
</style>