<template>
    <v-container>
        <h1>M-CHAT-R/F Cuestionario</h1>
        <v-form v-model="valid" ref="form">
            <v-card v-for="(question, index) in questions" :key="index" class="mb-4">
                <v-card-title>{{ question.text }}</v-card-title>
                <v-card-actions>
                    <v-radio-group v-model="answers[index]" :mandatory="true">
                        <v-radio label="Sí" :value="true"></v-radio>
                        <v-radio label="No" :value="false"></v-radio>
                    </v-radio-group>
                </v-card-actions>
            </v-card>
            <v-btn @click="calculateScore" color="success">Calcular Puntuación</v-btn>
        </v-form>
        <v-alert v-if="result" :type="result.type" class="mt-4">
            {{ result.message }}
        </v-alert>
    </v-container>
</template>

<script>

const SPECIAL_QUESTIONS = new Set([1, 4, 11]);

export default {
    data() {
        return {
            valid: false,
            answers: Array(20).fill(null),
            result: null,
            childAge: 0, // valor de edad del niño (cambiar en el back o no se como lo va a hacer)
            questions: [
                { text: 'Si usted señala algo al otro lado de la habitación, ¿su hijo/a lo mira?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Alguna vez se ha preguntado si su hijo/a es sordo/a?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a hace o realiza juegos de fantasía o imaginación?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿A su hijo le gusta subirse a cosas?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Hace su hijo/a movimientos inusuales con sus dedos cerca de sus ojos?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a señala con un dedo cuando quiere pedir algo o pedir ayuda?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a señala con un dedo cuando quiere mostrarle algo?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a se interesa en otros niños?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a le muestra cosas acercándolas o levantándolas para que usted las vea?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a responde cuando usted le llama por su nombre?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Cuándo usted sonríe a su hijo/a, él o ella también le sonríe?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Le molestan a su hijo/a ruidos cotidianos?', positiveAnswer: true, negativeAnswer: false },
                { text: '¿Su hijo/a camina solo?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Su hijo/a le mira a los ojos cuando usted le habla?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Su hijo/a imita sus movimientos?', positiveAnswer: false, negativeAnswer: true },
                { text: 'Si usted se gira a ver algo, ¿su hijo/a trata de mirar hacia lo que usted está mirando?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Su hijo/a intenta que usted le mire/preste atención?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Su hijo/a le entiende cuando usted le dice que haga algo?', positiveAnswer: false, negativeAnswer: true },
                { text: 'Si algo nuevo pasa, ¿su hijo/a le mira para ver como usted reacciona al respecto?', positiveAnswer: false, negativeAnswer: true },
                { text: '¿Le gustan a su hijo/a los juegos de movimiento?', positiveAnswer: false, negativeAnswer: true },
            ]
        };
    },
    methods: {
        calculateScore() {
            if (!this.allQuestionsAnswered()) {
                this.showWarning('Por favor, responda todas las preguntas antes de calcular la puntuación.');
                return;
            }

            const score = this.computeScore();
            this.determineRisk(score);
        },
        allQuestionsAnswered() {
            return this.answers.every(answer => answer !== null);
        },
        computeScore() {
            return this.answers.reduce((score, answer, index) => {
                return score + this.getScoreForAnswer(answer, index);
            }, 0);
        },
        getScoreForAnswer(answer, index) {
            if (SPECIAL_QUESTIONS.has(index)) {
                return answer === true ? 1 : 0;
            }
            return answer === false ? 1 : 0;
        },
        showWarning(message) {
            this.result = { type: 'warning', message };
        },
        determineRisk(score) { //validar edad del niño <24 meses
            if (this.childAge < 24 && score >= 0 && score <= 2) {
                this.result = { type: 'success', message: 'Bajo riesgo de TEA. Repetir M-CHAT-R a los 24 meses.' };
            } else if (score >= 0 && score <= 2) {
                this.result = { type: 'success', message: 'Bajo riesgo de TEA.' };
            } else if (score >= 3 && score <= 7) {
                this.result = { type: 'warning', message: 'Riesgo medio de TEA. Se recomienda entrevista de seguimiento.' };
            } else {
                this.result = { type: 'error', message: 'Alto riesgo de TEA. Remitir para evaluación diagnóstica inmediata.' };
            }
        },
    }

};
</script>
