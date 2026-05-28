<template>
    <div class="sidebar content-wrapper">
        <h3 class="title">Сигналы и статистика</h3>
        <div class="widgets">
            <div class="widgets__widget" v-if="volatility">
                <h4>Волатильность</h4>
                <h2>{{ volatility }} %</h2>
            </div>
            <div class="widgets__widget" v-if="predictions">
                <h4>Тренд</h4>
                <h2
                    :class="
                        trend > 0 ? 'increase' : trend < 0 ? 'decrease' : ''
                    "
                >
                    {{
                        trend > 0.5
                            ? "Бычий"
                            : trend < -0.5
                              ? "Медвежий"
                              : "Боковик"
                    }}
                </h2>
            </div>
            <!-- <div class="widgets__widget" v-if="ll"><h4>Волатильность</h4></div> -->
        </div>
        <div class="devider"></div>
        <h3 class="title">Прогноз за 20 шагов</h3>
        <div class="prob">
            <div class="prob-header">
                <h3 class="prob__title">Вероятность роста</h3>
                <div
                    class="prob__trend"
                    :class="
                        trend > 0 ? 'increase' : trend < 0 ? 'decrease' : ''
                    "
                >
                    {{ trend > 0 ? "+" : "" }}{{ trend }} %
                </div>
            </div>
            <table>
                <tr v-for="row in predictions">
                    <td>{{ row.method }}</td>
                    <td :class="row.change >= 0 ? 'increase' : 'decrease'">{{ row.value20.toFixed(2) }} ₽</td>
                </tr>
                <!-- <tr>
                    <td>Линейная</td>
                    <td>12 310 ₽</td>
                </tr>
                <tr>
                    <td>Скользящая</td>
                    <td>12 310 ₽</td>
                </tr> -->
            </table>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        volatility: { type: String },
        predictions: { type: Array },
    },
    computed: {
        trend() {
            let res = 0;
            this.predictions.forEach((item) => (res += Number(item.change)));
            return (res / this.predictions.length).toFixed(2);
        },
    },
};
</script>

<style lang="scss" scoped>
.sidebar {
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 15px;
    h3.title {
        font-size: 16px;
        text-transform: uppercase;
        color: var(--color-text-scnd);
    }
    .devider {
        width: 100%;
        height: 1px;
        background: var(--color-border);
    }
    .widgets {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 10px;
        &__widget {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            background: var(--color-bg);
            height: 70px;
            border-radius: 12px;
            padding: 5px 10px;
            h4 {
                font-size: 14px;
                color: var(--color-text-scnd);
            }
            h2 {
                font-size: 26px;
                &.increase {
                    color: var(--color-green);
                }
                &.decrease {
                    color: var(--color-red);
                }
            }
        }
    }
    .prob {
        padding: 8px;
        border-radius: 12px;
        border: 1px solid var(--color-border);
        &-header {
            display: flex;
            flex-direction: row;
            justify-content: space-between;
            align-items: center;
        }
        &__title {
            font-size: 16px;
        }
        &__trend {
            padding: 3px 12px;
            border-radius: 50vi;
            display: flex;
            align-items: center;
            gap: 5px;
            background: var(--color-page);
            &.increase {
                // background: var(--color-green-light);
                color: var(--color-green);
            }
            &.decrease {
                // background: var(--color-red-light);
                color: var(--color-red);
            }
        }
        table {
            width: 100%;
            tr td {
                padding: 10px 0;
                border-bottom: 1px solid var(--color-border);
                &:first-child {
                    color: var(--color-text-scnd);
                }
                &:last-child {
                    text-align: right;
                }
                &.increase{
                    color: var(--color-green);
                }
                &.decrease{
                    color: var(--color-red);
                }
            }
            tr:last-child td {
                border: none;
            }
        }
    }
}
</style>
