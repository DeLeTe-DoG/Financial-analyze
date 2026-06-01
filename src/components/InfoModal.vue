<template>
    <div class="modal-wrapper" v-if="visible" @click.self="closeModal()">
        <div class="modal content-wrapper">
            <table>
                <thead>
                    <tr>
                        <td>Метод</td>
                        <td>Суть метода</td>
                        <td>Сильные стороны</td>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="row in info">
                        <td>{{ row.method }}</td>
                        <td>{{ row.about }}</td>
                        <td>{{ row.advantages }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            info: [
                {
                    method: "Линейная регрессия",
                    about: "Строит оптимальную прямую линию через массив цен. Оценивает общий вектор движения.",
                    advantages:
                        "Игнорирует ложный шум, четко держит долгосрочный тренд.",
                },
                {
                    method: "Полиномиальная регрессия",
                    about: "Использует уравнение параболы. Находит изгибы графиков, улавливая ускорение или замедление.",
                    advantages:
                        "Заблаговременно видит затухание импульса и разворот.",
                },
                {
                    method: "ARIMA",
                    about: "Анализирует внутренние связи временного ряда: авторегрессию и шоки прошлых периодов.",
                    advantages:
                        "Высокая точность на циклических колебаниях внутри дня.",
                },
                {
                    method: "Скользящая регрессия",
                    about: "Постоянно пересчитывает коэффициенты, опираясь только на последнее «окно» свечей.",
                    advantages:
                        "Мгновенно реагирует на резкие ценовые развороты.",
                },
            ],
        };
    },
    methods: {
        showModal() {
            this.visible = true
        },
        closeModal() {
            this.visible = false
        },
    }
};
</script>

<style scoped lang="scss">
.modal-wrapper {
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.4);
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    align-items: center;
    justify-content: center;
}
.modal {
    width: 800px;
    max-width: 100vw;
    table {
        tr:last-child td{
            border: none;
        }
        td {
            padding: 15px;
            border-bottom: 1px solid var(--color-border);
        }
    }
    thead td {
        color: var(--color-text-scnd);
        text-transform: uppercase;
    }
}
</style>
