<template>
    <header class="header content-wrapper">
        <h1 class="header-title">FinAn</h1>
        <select
            name="shareName"
            id="shareName"
            :value="activeShare"
            class="header-select select"
            @change="$emit('handleShare', $event.target.value)"
        >
            <option :value="item.value" v-for="item in shareList">
                {{ item.label }}
            </option>
        </select>
        <button class="btn">
            О методах
            <Icon variant="info" />
        </button>
    </header>
</template>

<script>
import axios from "axios";
import Icon from "./Icon.vue";

export default {
    props: {
        shareList: {type: Array},
        activeShare: {type: String},
    },
    emits: ["handleShare"],
    components: { Icon },
    methods: {
        // async getShares() {
        //     try {
        //         const response = await axios.get('https://iss.moex.com/iss/engines/stock/markets/shares/securities.json')
        //         console.log(response)
        //     } catch (error) {
        //         console.log(error)
        //     }
        // }
    },
};
</script>

<style lang="scss">
.header {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 15px;
    .btn {
        margin-left: auto;
    }
    &-title {
        font-size: 24px;
    }
}
</style>
