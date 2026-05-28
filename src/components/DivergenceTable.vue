<template>
    <div class="content-wrapper divergence-table">
        <h3 class="divergence-table__title">Таблица расхождений моделей</h3>
        <table
            v-if="
                predictions.filter((row) => row.method.includes('ARIMA')).length
            "
        >
            <thead>
                <tr>
                    <td>Метод</td>
                    <td>Прогноз через 1 шаг</td>
                    <td>Прогноз через 10 шагов</td>
                    <td>Прогноз через 20 шагов</td>
                    <td>Направление</td>
                </tr>
            </thead>
            <tbody>
                <tr
                    v-for="row in predictions.filter((row) =>
                        row.method.includes('ARIMA'),
                    )"
                >
                    <td>{{ row.method }}</td>
                    <td>{{ row.value1 }}</td>
                    <td>{{ row.value10 }}</td>
                    <td>{{ row.value20 }}</td>
                    <td
                        :class="
                            row.change > 0.5 ? 'increase' : row.change < 0.5 ? 'decrease' : ''
                        "
                    >
                        {{
                            row.change > 0.5
                                ? "Рост"
                                : row.change < -0.5
                                  ? "Падение"
                                  : "Боковик"
                        }}
                    </td>
                </tr>
            </tbody>
        </table>
        <h2 class="no-data" v-else>
            Для построения таблицы необходимо использовать метод ARIMA
        </h2>
    </div>
</template>

<script>
export default {
    props: {
        predictions: {
            type: Array,
        },
    },
};
</script>

<style lang="scss" scoped>
.no-data {
    margin: 50px auto;
    color: var(--color-text-scnd);
    font-size: 20px;
    text-align: center;
}
table{
    width: 100%;
    border-spacing: 0;
    thead td{
        background: var(--color-border);
        &:first-child{
            border-radius: 12px 0 0 12px;
        }
        &:last-child{
            border-radius: 0 12px 12px 0;
        }
    }
    tr td{
        padding: 15px 10px;
        border-bottom: 1px solid var(--color-border);
        &.increase{
            color: var(--color-green);
        }
        &.decrease{
            color: var(--color-red);
        }
    }
}
</style>
