<template>
    <Header
        :activeShare="activeShare"
        :shareList="shareList"
        @handleShare="applyFilters($event)"
    />
    <div class="filters">
        <div class="filters-methods">
            <p>Методы:</p>
            <div
                class="filters-methods__method"
                :class="{ active: method.active }"
                @click="method.active = !method.active"
                v-for="method in chartMethods"
            >
                {{ method.label }}
            </div>
        </div>
        <select
            name=""
            id=""
            class="filters-intervals select"
            v-model="activeInterval"
        >
            <option :value="item.value" v-for="item in intervals">
                {{ item.label }}
            </option>
        </select>
        <button class="btn" @click="applyFilters()">Применить</button>
    </div>
    <main class="main">
        <div class="main-column">
            <div class="line-chart content-wrapper">
                <Line v-if="dataLoaded" id="main-line" :data="chartData" />
            </div>
            <DivergenceTable :predictions="predictionTrends" />
        </div>
        <Sidebar :volatility="volatility" :predictions="predictionTrends" />
    </main>
</template>

<script>
import Header from "./components/Header.vue";
import axios from "axios";
import Sidebar from "./components/Sidebar.vue";
import DivergenceTable from "./components/DivergenceTable.vue";

import { Line } from "vue-chartjs";
import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    LineElement,
    CategoryScale,
    LinearScale,
    PointElement,
} from "chart.js";
ChartJS.register(
    Title,
    Tooltip,
    Legend,
    LineElement,
    CategoryScale,
    LinearScale,
    PointElement,
);

export default {
    components: { Header, Line, Sidebar, DivergenceTable },
    data() {
        return {
            chartData: {
                labels: [],
                datasets: [],
            },
            dataLoaded: false,
            volatility: null,
            chartMethods: [
                {
                    id: 1,
                    label: "Линейная регр.",
                    value: "linear",
                    active: true,
                },
                {
                    id: 2,
                    label: "Полиномиальная регр.",
                    value: "polynom",
                    active: false,
                },
                {
                    id: 3,
                    label: "ARIMA",
                    value: "arima",
                    active: false,
                },
                {
                    id: 4,
                    label: "Скользящая линейная",
                    value: "moving",
                    active: false,
                },
            ],
            intervals: [
                {
                    id: 1,
                    label: "1д",
                    value: "24",
                },
                {
                    id: 2,
                    label: "1ч",
                    value: "60",
                },
            ],
            activeInterval: "60",

            activeShare: "SBER",
            shareList: [
                {
                    id: 1,
                    label: "СберБанк",
                    value: "SBER",
                },
                {
                    id: 2,
                    label: "Аэрофлот",
                    value: "AFLT",
                },
                {
                    id: 3,
                    label: "Норильский Никель",
                    value: "GMKN",
                },
                {
                    id: 4,
                    label: "Газпром",
                    value: "GAZP",
                },
            ],
            predictionTrends: [
                // {
                //     method: '...',
                //     value: '...',
                //     change: '...'
                // },
            ],
        };
    },
    methods: {
        setNDays(m) {
            // t + m days
            const today = new Date();
            for (let i = 1; i < m; i++) {
                let dateI = new Date();
                dateI.setDate(dateI.getDate() + i);

                const year = dateI.getFullYear();
                let month = String(dateI.getMonth() + 1);
                let day = String(dateI.getDate());

                month.length == 1 ? (month = `0${month}`) : (month = month);
                day.length == 1 ? (day = `0${day}`) : (day = day);
                dateI = `${year}-${month}-${day}`;
                this.chartData.labels.push(dateI);
            }
        },
        async getAssets(share) {
            this.dataLoaded = false;
            let date = new Date();
            date.setDate(
                date.getDate() - (this.activeInterval == "24" ? 60 : 10),
            );
            console.log(date);
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2, "0");
            const day = String(date.getUTCDate()).padStart(2, "0");
            await axios
                .get(
                    `https://iss.moex.com/iss/engines/stock/markets/shares/securities/${this.activeShare}/candles.json?from=${year}-${month}-${day}&interval=${this.activeInterval}`,
                )
                .then((response) => {
                    console.log(response.data);

                    const data = response.data.candles.data;
                    const closeCosts = data.map((item) => item[1]);
                    let xAxis = [];
                    for (let i = 0; i < data.length; i++) {
                        const itemDate = data[i][6].split(" ")[0];
                        if (!xAxis.includes(itemDate)) {
                            xAxis.push(itemDate);
                        } else {
                            xAxis.push("");
                        }
                    }
                    console.log(closeCosts);
                    console.log(xAxis);
                    this.chartData.datasets.push({
                        label: "history data",
                        data: closeCosts,
                        borderColor: "#f50000",
                        pointRadius: 3,
                        pointBorderColor: "#f50",
                    });
                    this.chartData.labels = xAxis;
                    this.setNDays(20);
                    return closeCosts;
                })
                .then((data) => {
                    this.chartMethods.forEach((method) => {
                        if (method.active) {
                            if (method.value == "linear") {
                                let linearRegData = Array();
                                linearRegData.push(
                                    ...this.linearRegression(data),
                                );
                                this.chartData.datasets.push({
                                    label: method.label,
                                    data: linearRegData,
                                    pointRadius: 3,
                                    borderColor: "#969696",
                                });
                            }
                            if (method.value == "polynom") {
                                let polynomialRegData = Array(
                                    data.length - 1,
                                ).fill(null);
                                polynomialRegData.push(data[data.length - 1]);
                                polynomialRegData.push(
                                    ...this.polynomialRegression(data, 2, 20),
                                );
                                this.chartData.datasets.push({
                                    label: method.label,
                                    data: polynomialRegData,
                                    pointRadius: 3,
                                    borderColor: "#009500",
                                });
                            }
                            if (method.value == "arima") {
                                this.chartData.datasets.push({
                                    label: "ARIMA 1",
                                    data: [
                                        ...data,
                                        ...this.simpleArimaPredict(data)
                                            .forecast,
                                    ],
                                    pointRadius: 3,
                                    borderColor: "#15D2E3",
                                });
                                this.chartData.datasets.push({
                                    label: "ARIMA 2",
                                    data: [
                                        ...data,
                                        ...this.simpleArimaPredict(data)
                                            .forecast,
                                    ],
                                    pointRadius: 3,
                                    borderColor: "#0F939F",
                                });
                                this.chartData.datasets.push({
                                    label: "ARIMA 3",
                                    data: [
                                        ...data,
                                        ...this.simpleArimaPredict(data)
                                            .forecast,
                                    ],
                                    pointRadius: 3,
                                    borderColor: "#0D7A84",
                                });
                            }
                            if (method.value == "moving") {
                                let movingRegData = Array(data.length - 1).fill(
                                    null,
                                );
                                movingRegData.push(data[data.length - 1]);
                                movingRegData.push(
                                    ...this.movingLinearRegression(data),
                                );
                                this.chartData.datasets.push({
                                    label: method.label,
                                    data: movingRegData,
                                    pointRadius: 3,
                                    borderColor: "#000095",
                                });
                            }
                        }
                    });
                    this.simpleArimaPredict(data);
                    console.log(this.chartData);

                    this.predictionTrends = this.chartData.datasets.filter(
                        (item) => item.label != "history data",
                    );
                    this.predictionTrends = this.predictionTrends.map(
                        (item) => {
                            const costNow = data[data.length - 1];
                            const change =
                                (-(costNow - item.data[item.data.length - 1]) /
                                    costNow) *
                                100;
                            return {
                                method: item.label,
                                value1: item.data[item.data.length - 20],
                                value10: item.data[item.data.length - 10],
                                value20: item.data[item.data.length - 1],
                                change: change,
                            };
                        },
                    );
                    console.log(this.predictionTrends);
                });
            this.dataLoaded = true;
        },
        // Yt = A + B * t
        linearRegression(y, steps = 20) {
            const n = y.length;
            let sumX = 0,
                sumY = 0,
                sumXY = 0,
                sumXX = 0;
            for (let t = 0; t < n; t++) {
                sumX += t; // ∑t (from 0 to n)
                sumY += y[t]; // ∑Yt (from 0 to n)
                sumXY += t * y[t]; // ∑Yt * t (from 0 to n)
                sumXX += t * t; // ∑t^2 (from 0 to n)
            }
            const b = (n * sumXY - sumX * sumY) / (n * sumXX - sumX * sumX);
            const a = (sumY - b * sumX) / n;
            // return { a, b };
            // const forecast = a + b * y.length
            // console.log(forecast)
            let forecasts = [];
            for (let i = 0; i < n + steps; i++) {
                const t = n + i;
                forecasts.push(a + b * t);
            }
            return forecasts;
        },
        polynomialRegression(y, degree = 2, steps = 20) {
            const n = y.length;

            // считаем суммы степеней X
            let sumX = Array(2 * degree + 1).fill(0);
            for (let i = 0; i < n; i++) {
                for (let j = 0; j <= 2 * degree; j++) {
                    sumX[j] += Math.pow(i, j);
                }
            }

            // считаем суммы X^k * Y
            let sumXY = Array(degree + 1).fill(0);
            for (let i = 0; i < n; i++) {
                for (let j = 0; j <= degree; j++) {
                    sumXY[j] += Math.pow(i, j) * y[i];
                }
            }

            // строим матрицу системы
            let A = [];
            for (let i = 0; i <= degree; i++) {
                A[i] = [];
                for (let j = 0; j <= degree; j++) {
                    A[i][j] = sumX[i + j];
                }
            }

            let B = sumXY;

            // 🔧 Решение системы (метод Гаусса)
            function solve(A, B) {
                let n = B.length;

                for (let i = 0; i < n; i++) {
                    // поиск максимума (устойчивость)
                    let maxRow = i;
                    for (let k = i + 1; k < n; k++) {
                        if (Math.abs(A[k][i]) > Math.abs(A[maxRow][i])) {
                            maxRow = k;
                        }
                    }

                    // swap
                    [A[i], A[maxRow]] = [A[maxRow], A[i]];
                    [B[i], B[maxRow]] = [B[maxRow], B[i]];

                    // нормализация
                    let diag = A[i][i];
                    for (let j = i; j < n; j++) A[i][j] /= diag;
                    B[i] /= diag;

                    // обнуление
                    for (let k = 0; k < n; k++) {
                        if (k === i) continue;
                        let factor = A[k][i];
                        for (let j = i; j < n; j++) {
                            A[k][j] -= factor * A[i][j];
                        }
                        B[k] -= factor * B[i];
                    }
                }

                return B; // коэффициенты
            }

            const coeffs = solve(A, B);

            // прогноз
            let forecasts = [];

            for (let i = 0; i < steps; i++) {
                let t = n + i;
                let val = 0;

                for (let d = 0; d <= degree; d++) {
                    val += coeffs[d] * Math.pow(t, d);
                }

                forecasts.push(val);
            }

            return forecasts;
        },
        movingLinearRegression(y, windowSize = 10, steps = 20) {
            const n = y.length;

            // берём последние windowSize точек
            const start = Math.max(0, n - windowSize);
            const slice = y.slice(start);

            const m = slice.length;

            let sumX = 0,
                sumY = 0,
                sumXY = 0,
                sumXX = 0;

            // считаем регрессию по окну
            for (let i = 0; i < m; i++) {
                let x = i;
                let val = slice[i];

                sumX += x;
                sumY += val;
                sumXY += x * val;
                sumXX += x * x;
            }

            const b = (m * sumXY - sumX * sumY) / (m * sumXX - sumX * sumX);
            const a = (sumY - b * sumX) / m;

            // --- прогноз ---
            let forecasts = [];

            for (let i = 0; i < steps; i++) {
                let t = m + i;
                forecasts.push(a + b * t);
            }

            // --- убираем скачок ---
            const lastReal = y[n - 1];
            const offset = lastReal - forecasts[0];

            forecasts = forecasts.map((v) => v + offset);

            return forecasts;
        },
        simpleArimaPredict(data, steps = 20) {
            if (data.length < 2)
                return { forecast: [], volatility: 0, volatilityPct: 0 };

            // 1. Считаем изменения (разницы)
            const diffs = [];
            for (let i = 1; i < data.length; i++) {
                diffs.push(data[i] - data[i - 1]);
            }

            // 2. Расчет волатильности (Стандартное отклонение)
            const n = diffs.length;
            const avgDiff = diffs.reduce((a, b) => a + b, 0) / n;

            // Сумма квадратов отклонений
            const squareDiffs = diffs.map((d) => Math.pow(d - avgDiff, 2));
            const avgSquareDiff = squareDiffs.reduce((a, b) => a + b, 0) / n;

            // Итоговая волатильность (сигма)
            const volatility = Math.sqrt(avgSquareDiff);

            // Волатильность в процентах относительно последней цены
            const lastPrice = data[data.length - 1];
            const volatilityPct = (volatility / lastPrice) * 100;

            // 3. Генерация прогноза (тот же алгоритм с шумом)
            const forecast = [];
            let currentPrice = lastPrice;
            let lastDiff = diffs[diffs.length - 1];

            const gaussianRandom = () => {
                let u = Math.random(),
                    v = Math.random();
                return (
                    Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v)
                );
            };

            for (let i = 0; i < steps; i++) {
                const noise = gaussianRandom() * volatility;
                const nextDiff = avgDiff + (lastDiff - avgDiff) * 0.5 + noise;
                currentPrice += nextDiff;
                forecast.push(Number(currentPrice.toFixed(2)));
                lastDiff = nextDiff;
            }

            this.volatility = (volatilityPct * 100).toFixed(1);
            console.log(volatilityPct);

            return {
                forecast: forecast,
                volatility: volatility.toFixed(2), // В единицах валюты
                volatilityPct: volatilityPct.toFixed(2), // В процентах
            };
        },

        applyFilters(share) {
            if (share) {
                this.activeShare = share;
            }
            this.chartData = {
                label: [],
                datasets: [],
            };
            this.getAssets(share);
        },
    },
    mounted() {
        this.getAssets(this.activeShare);
    },
};
</script>

<style lang="scss" scoped>
.line-chart {
    width: 100%;
    margin: 0 auto;
}
.filters {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    padding: 20px 0;
    gap: 10px;
    &-methods {
        display: flex;
        flex-direction: row;
        align-items: center;
        flex-wrap: wrap;
        gap: 10px;
        p {
            color: var(--color-text-scnd);
        }
        &__method {
            padding: 5px 15px;
            border: 1px solid var(--color-border);
            border-radius: 50vi;
            transition: all 0.1s ease-in;
            cursor: pointer;
            &.active {
                background: var(--color-light-blue);
                color: var(--color-blue);
                border-color: var(--color-blue);
            }
        }
    }
    &-intervals {
        margin-left: auto;
    }
}
</style>
