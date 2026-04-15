<template>
    <Header />
    <!-- <form action="" class="choose-assets">
        <label for="assets" class="choose-asset__title"></label>
        <select name="assets" id="assets" class="choose-asset__select">
            <option value=""></option>
        </select>
    </form> -->
    {{ volatility < 1 ? 'Прогнозирование может быть достаточно точным' : volatility > 3 ? 'Прогноз имеет умеренный риск' : 'Прогноз имеет высокий риск' }}
    <div class="line-chart">
        <Line v-if="dataLoaded" id="main-line" :data="chartData" />
    </div>
</template>

<script>
import Header from "./components/Header.vue";
import axios from "axios";

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
    components: { Header, Line },
    data() {
        return {
            chartData: {
                labels: [],
                datasets: [],
            },
            dataLoaded: false,
            volatility: null,
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
        async getAssets() {
            this.dataLoaded = false;
            await axios
                .get(
                    "https://iss.moex.com/iss/engines/stock/markets/shares/securities/SBER/candles.json?from=2026-02-01&interval=24",
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
                    let linearRegData = Array(data.length - 1).fill(null);
                    linearRegData.push(data[data.length - 1]);
                    let polynomialRegData = [...linearRegData];
                    let movingRegData = [...linearRegData];
                    let ARIMAData = [...linearRegData];

                    linearRegData.push(...this.linearRegression(data));

                    polynomialRegData.push(
                        ...this.polynomialRegression(data, 2, 20),
                    );
                    movingRegData.push(...this.movingLinearRegression(data));

                    this.chartData.datasets.push({
                        label: "linear regression",
                        data: linearRegData,
                        pointRadius: 3,
                        borderColor: "#969696",
                    });
                    this.chartData.datasets.push({
                        label: "polynomal regression",
                        data: polynomialRegData,
                        pointRadius: 3,
                        borderColor: "#009500",
                    });
                    this.chartData.datasets.push({
                        label: "moving regression",
                        data: movingRegData,
                        pointRadius: 3,
                        borderColor: "#000095",
                    });
                    this.chartData.datasets.push({
                        label: "ARIMA 1",
                        data: [...data, ...this.simpleArimaPredict(data).forecast],
                        pointRadius: 3,
                        borderColor: "#15D2E3",
                    });
                    this.chartData.datasets.push({
                        label: "ARIMA 2",
                        data: [...data, ...this.simpleArimaPredict(data).forecast],
                        pointRadius: 3,
                        borderColor: "#0F939F",
                    });
                    this.chartData.datasets.push({
                        label: "ARIMA 3",
                        data: [...data, ...this.simpleArimaPredict(data).forecast],
                        pointRadius: 3,
                        borderColor: "#0D7A84",
                    });
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
            for (let i = 0; i < steps; i++) {
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
        // Функция ARIMA(1,1,1) на чистом JS
        forecastARIMA(prices, steps = 20) {
            // 1. Дифференцируем ряд (d=1)
            let diff = [];
            for (let i = 1; i < prices.length; i++) {
                diff.push(prices[i] - prices[i - 1]);
            }

            // 2. Находим коэффициент AR(1) через метод наименьших квадратов
            // diff[t] = phi * diff[t-1] + error
            let n = diff.length;
            let sumX = 0,
                sumY = 0,
                sumXY = 0,
                sumXX = 0;
            for (let i = 1; i < n; i++) {
                let x = diff[i - 1];
                let y = diff[i];
                sumX += x;
                sumY += y;
                sumXY += x * y;
                sumXX += x * x;
            }
            let phi =
                (sumXY - (sumX * sumY) / (n - 1)) /
                (sumXX - (sumX * sumX) / (n - 1));

            // 3. Находим MA(1) коэффициент theta через простой метод (автокорреляция)
            let errors = [];
            for (let i = 1; i < n; i++) {
                errors.push(diff[i] - phi * diff[i - 1]);
            }
            // theta = корреляция diff[t] и errors[t-1]
            let sumXY2 = 0,
                sumXX2 = 0;
            for (let i = 1; i < errors.length; i++) {
                sumXY2 += diff[i] * errors[i - 1];
                sumXX2 += errors[i - 1] ** 2;
            }
            let theta = sumXY2 / sumXX2;

            // 4. Прогнозируем следующий diff
            let lastDiff = diff[diff.length - 1];
            let lastError = errors[errors.length - 1];
            let forecast = [];
            let lastPrice = prices[prices.length - 1];

            for (let i = 0; i < steps; i++) {
                let nextDiff = phi * lastDiff + theta * lastError; // ARMA(1,1)
                let nextPrice = lastPrice + nextDiff; // Интегрируем обратно
                forecast.push(nextPrice);

                // Обновляем переменные
                lastDiff = nextDiff;
                lastError = nextDiff - phi * lastDiff; // простая ошибка
                lastPrice = nextPrice;
            }

            return forecast;
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

            this.volatility = volatilityPct

            return {
                forecast: forecast,
                volatility: volatility.toFixed(2), // В единицах валюты
                volatilityPct: volatilityPct.toFixed(2), // В процентах
            };
        },

        // Пример использования
        // let prices = [100, 102, 101, 105, 107, 110, 108, 112];
        // let predicted = forecastARIMA(prices, 5);
        // console.log(predicted);
    },
    mounted() {
        this.getAssets();
    },
};
</script>

<style lang="scss" scoped>
.line-chart {
    width: 90%;
    margin: 0 auto;
}
</style>
