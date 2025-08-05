<template>
  <div class="health-quiz-container">
    <div class="quiz-header">
      <h3>Health Assessment Quiz</h3>
      <p>Answer these questions to see how your lifestyle affects your organ health</p>
      <div v-if="hasSavedData" class="saved-data-indicator">
        💾 You have saved health data that will be applied to the body map
      </div>
    </div>

    <div v-if="!quizCompleted" class="quiz-content">
        <div class="question-progress">
          <el-progress 
            :percentage="progressPercentage" 
            :stroke-width="6"
            color="#8E4EC6"
          />
          <span class="progress-text">Question {{ currentQuestionIndex + 1 }} of {{ questions.length }}</span>
        </div>

        <div class="current-question">
          <h4>{{ currentQuestion.question }}</h4>
          
          <div v-if="currentQuestion.type === 'number'" class="number-input">
            <el-input-number
              v-model="currentAnswer"
              :min="currentQuestion.min"
              :max="currentQuestion.max"
              :step="currentQuestion.step || 1"
              size="large"
            />
            <span class="unit">{{ currentQuestion.unit }}</span>
          </div>

          <div v-else-if="currentQuestion.type === 'select'" class="select-input">
            <el-select 
              v-model="currentAnswer" 
              placeholder="Select an option"
              size="large"
              style="width: 100%"
            >
              <el-option
                v-for="option in currentQuestion.options"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
          </div>

          <div v-else-if="currentQuestion.type === 'radio'" class="radio-input">
            <el-radio-group v-model="currentAnswer" size="large">
              <el-radio 
                v-for="option in currentQuestion.options"
                :key="option.value"
                :label="option.value"
                class="radio-option"
              >
                {{ option.label }}
              </el-radio>
            </el-radio-group>
          </div>
        </div>

        <div class="quiz-navigation">
          <el-button 
            v-if="currentQuestionIndex > 0"
            @click="previousQuestion"
          >
            ← Previous
          </el-button>
          
          <el-button 
            v-if="currentQuestionIndex < questions.length - 1"
            @click="nextQuestion"
            type="primary"
            :disabled="currentAnswer === null"
            class="purple-button"
          >
            Next →
          </el-button>
          
          <el-button 
            v-if="currentQuestionIndex === questions.length - 1"
            @click="completeQuiz"
            type="success"
            :disabled="currentAnswer === null"
            class="purple-button"
          >
            ✓ Complete Quiz
          </el-button>
        </div>
      </div>

      <div v-else class="quiz-results">
        <div class="results-header">
          <div class="success-icon">✓</div>
          <h3>Quiz Complete!</h3>
          <p>Your health assessment has been calculated based on your answers.</p>
        </div>

        <div class="health-summary">
          <h4>Your Organ Health Summary:</h4>
          <div class="organ-health-grid">
            <div 
              v-for="(health, organ) in calculatedHealth" 
              :key="organ"
              class="organ-health-item"
            >
              <span class="organ-name">{{ getOrganDisplayName(organ) }}</span>
              <el-progress 
                :percentage="health" 
                :color="getHealthProgressColor(health)"
                :stroke-width="8"
              />
              <span class="health-value">{{ health }}%</span>
            </div>
          </div>
        </div>

        <div class="quiz-actions">
          <el-button @click="resetQuiz">🔄 Retake Quiz</el-button>
          <el-button @click="applyHealthData" type="primary" class="purple-button">
            📤 Apply to Body Map
          </el-button>
        </div>
      </div>
  </div>
</template>

<script>
export default {
  name: 'HealthQuiz',
  data() {
    return {
      currentQuestionIndex: 0,
      currentAnswer: null,
      answers: {},
      quizCompleted: false,
      calculatedHealth: {},
      hasSavedData: false,
      questions: [
        {
          id: 'exercise',
          question: 'How many hours do you exercise each week?',
          type: 'number',
          min: 0,
          max: 20,
          step: 0.5,
          unit: 'hours',
          affects: ['heart', 'lungs', 'circulatory', 'brain']
        },
        {
          id: 'pm25',
          question: 'What is the average PM 2.5 air pollution level at your location?',
          type: 'select',
          unit: 'μg/m³',
          affects: ['lungs', 'heart', 'brain'],
          options: [
            { label: 'Excellent (0-10 μg/m³)', value: 5 },
            { label: 'Good (10-20 μg/m³)', value: 15 },
            { label: 'Moderate (20-35 μg/m³)', value: 25 },
            { label: 'Poor (35-55 μg/m³)', value: 45 },
            { label: 'Very Poor (55+ μg/m³)', value: 65 }
          ]
        },
        {
          id: 'plastic_containers',
          question: 'How many plastic takeaway containers do you use each week?',
          type: 'number',
          min: 0,
          max: 20,
          unit: 'containers',
          affects: ['liver', 'stomach', 'intestine-small', 'intestine-large']
        },
        {
          id: 'sleep',
          question: 'How many hours of sleep do you get per night on average?',
          type: 'number',
          min: 3,
          max: 12,
          step: 0.5,
          unit: 'hours',
          affects: ['brain', 'heart', 'liver']
        },
        {
          id: 'diet',
          question: 'How would you describe your diet?',
          type: 'radio',
          affects: ['stomach', 'liver', 'intestine-small', 'intestine-large', 'heart'],
          options: [
            { label: 'Mostly whole foods, lots of vegetables', value: 90 },
            { label: 'Balanced with some processed foods', value: 75 },
            { label: 'Mixed diet, regular fast food', value: 60 },
            { label: 'Mostly processed and fast foods', value: 40 }
          ]
        },
        {
          id: 'stress',
          question: 'How would you rate your stress levels?',
          type: 'radio',
          affects: ['brain', 'heart', 'stomach', 'liver'],
          options: [
            { label: 'Very low stress', value: 95 },
            { label: 'Low stress', value: 85 },
            { label: 'Moderate stress', value: 70 },
            { label: 'High stress', value: 55 },
            { label: 'Very high stress', value: 40 }
          ]
        },
        {
          id: 'smoking',
          question: 'Do you smoke or vape?',
          type: 'radio',
          affects: ['lungs', 'heart', 'brain'],
          options: [
            { label: 'Never', value: 100 },
            { label: 'Quit over 5 years ago', value: 85 },
            { label: 'Quit 1-5 years ago', value: 75 },
            { label: 'Quit within last year', value: 65 },
            { label: 'Occasionally (social/light)', value: 50 },
            { label: 'Regular smoker/vaper', value: 30 }
          ]
        }
      ]
    }
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    progressPercentage() {
      return Math.round(((this.currentQuestionIndex + 1) / this.questions.length) * 100);
    }
  },
  watch: {
    currentQuestionIndex() {
      // Load existing answer if available
      this.currentAnswer = this.answers[this.currentQuestion.id] || null;
    }
  },
  methods: {
    nextQuestion() {
      this.saveCurrentAnswer();
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
      }
    },
    previousQuestion() {
      this.saveCurrentAnswer();
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--;
      }
    },
    saveCurrentAnswer() {
      if (this.currentAnswer !== null) {
        this.answers[this.currentQuestion.id] = this.currentAnswer;
      }
    },
    completeQuiz() {
      this.saveCurrentAnswer();
      this.calculateHealth();
      this.quizCompleted = true;
      
      // Save to local storage immediately when quiz is completed
      this.saveToLocalStorage();
      
      // Emit the calculated health data to parent component
      this.$emit('health-data-updated', this.calculatedHealth);
      this.$emit('quiz-completed', {
        answers: this.answers,
        healthData: this.calculatedHealth
      });
    },
    calculateHealth() {
      // Base health scores
      const baseHealth = {
        'brain': 80,
        'heart': 80,
        'lungs': 80,
        'liver': 80,
        'stomach': 80,
        'intestine-small': 80,
        'intestine-large': 80
      };

      // Apply modifiers based on answers
      Object.keys(baseHealth).forEach(organ => {
        let healthScore = baseHealth[organ];

        // Exercise effects
        if (this.answers.exercise !== undefined) {
          const exercise = this.answers.exercise;
          if (['heart', 'lungs', 'brain'].includes(organ)) {
            healthScore += Math.min(exercise * 2, 20); // Up to +20 for 10+ hours
          }
        }

        // PM 2.5 effects
        if (this.answers.pm25 !== undefined) {
          const pm25 = this.answers.pm25;
          if (['lungs', 'heart', 'brain'].includes(organ)) {
            healthScore -= Math.max((pm25 - 10) * 0.5, 0); // Penalty for high PM 2.5
          }
        }

        // Plastic container effects
        if (this.answers.plastic_containers !== undefined) {
          const plastic = this.answers.plastic_containers;
          if (['liver', 'stomach', 'intestine-small', 'intestine-large'].includes(organ)) {
            healthScore -= plastic * 2; // -2 per container
          }
        }

        // Sleep effects
        if (this.answers.sleep !== undefined) {
          const sleep = this.answers.sleep;
          if (['brain', 'heart', 'liver'].includes(organ)) {
            const sleepScore = Math.abs(sleep - 8); // Optimal is 8 hours
            healthScore -= sleepScore * 3;
          }
        }

        // Diet effects
        if (this.answers.diet !== undefined) {
          const dietScore = this.answers.diet;
          if (['stomach', 'liver', 'intestine-small', 'intestine-large', 'heart'].includes(organ)) {
            healthScore = (healthScore + dietScore) / 2; // Average with diet score
          }
        }

        // Stress effects
        if (this.answers.stress !== undefined) {
          const stressScore = this.answers.stress;
          if (['brain', 'heart', 'stomach', 'liver'].includes(organ)) {
            healthScore = (healthScore + stressScore) / 2;
          }
        }

        // Smoking effects
        if (this.answers.smoking !== undefined) {
          const smokingScore = this.answers.smoking;
          if (['lungs', 'heart', 'brain'].includes(organ)) {
            healthScore = (healthScore + smokingScore) / 2;
          }
        }

        // Ensure score is between 0 and 100
        this.calculatedHealth[organ] = Math.max(0, Math.min(100, Math.round(healthScore)));
      });
    },
    getOrganDisplayName(organ) {
      const names = {
        'brain': 'Brain',
        'heart': 'Heart',
        'lungs': 'Lungs',
        'liver': 'Liver',
        'stomach': 'Stomach',
        'intestine-small': 'Small Intestine',
        'intestine-large': 'Large Intestine'
      };
      return names[organ] || organ;
    },
    getHealthProgressColor(health) {
      if (health >= 90) return '#22c55e';
      if (health >= 75) return '#f59e0b';
      if (health >= 60) return '#f97316';
      return '#ef4444';
    },
    resetQuiz() {
      this.currentQuestionIndex = 0;
      this.currentAnswer = null;
      this.answers = {};
      this.quizCompleted = false;
      this.calculatedHealth = {};
      this.hasSavedData = false;
      
      // Optionally clear local storage when resetting
      this.clearLocalStorage();
    },
    applyHealthData() {
      // Data is already saved to local storage in completeQuiz()
      // Just emit events to notify parent component
      this.$emit('health-data-updated', this.calculatedHealth);
      this.$emit('quiz-completed', {
        answers: this.answers,
        healthData: this.calculatedHealth
      });
    },
    saveToLocalStorage() {
      const healthData = {
        timestamp: new Date().toISOString(),
        answers: this.answers,
        organHealth: this.calculatedHealth
      };
      
      try {
        localStorage.setItem('healthQuizResults', JSON.stringify(healthData));
        console.log('Health quiz results saved to local storage');
      } catch (error) {
        console.error('Failed to save health quiz results:', error);
      }
    },
    loadFromLocalStorage() {
      try {
        const savedData = localStorage.getItem('healthQuizResults');
        if (savedData) {
          const healthData = JSON.parse(savedData);
          return healthData.organHealth || null;
        }
      } catch (error) {
        console.error('Failed to load health quiz results:', error);
      }
      return null;
    },
    clearLocalStorage() {
      try {
        localStorage.removeItem('healthQuizResults');
        console.log('Health quiz results cleared from local storage');
      } catch (error) {
        console.error('Failed to clear health quiz results:', error);
      }
    }
  },
  mounted() {
    // Initialize with first question
    this.currentAnswer = this.answers[this.currentQuestion.id] || null;
    
    // Check if there are saved results and emit them to parent
    const savedHealth = this.loadFromLocalStorage();
    if (savedHealth) {
      this.hasSavedData = true;
      console.log('Found saved health data:', savedHealth);
      this.$emit('health-data-loaded', savedHealth);
    }
  }
}
</script>

<style scoped lang="scss">
.health-quiz-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.quiz-card {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.quiz-header {
  text-align: center;
  
  h3 {
    margin: 0 0 8px 0;
    color: #2c3e50;
  }
  
  p {
    margin: 0;
    color: #666;
    font-size: 14px;
  }
  
  .saved-data-indicator {
    margin-top: 12px;
    padding: 8px 12px;
    background-color: rgba(142, 78, 198, 0.1);
    border: 1px solid #8E4EC6;
    border-radius: 4px;
    color: #7b2cbf;
    font-size: 13px;
    font-weight: 500;
  }
}

.quiz-content {
  .question-progress {
    margin-bottom: 32px;
    
    .progress-text {
      display: block;
      text-align: center;
      margin-top: 8px;
      font-size: 14px;
      color: #666;
    }
  }
  
  .current-question {
    margin-bottom: 32px;
    
    h4 {
      margin-bottom: 20px;
      color: #2c3e50;
      font-size: 18px;
      line-height: 1.4;
    }
    
    .number-input {
      display: flex;
      align-items: center;
      gap: 12px;
      
      .unit {
        font-weight: 500;
        color: #666;
      }
    }
    
    .radio-input {
      .radio-option {
        display: block;
        margin-bottom: 12px;
        line-height: 1.4;
      }
    }
  }
  
  .quiz-navigation {
    display: flex;
    justify-content: space-between;
    gap: 12px;
  }
}

.quiz-results {
  .results-header {
    text-align: center;
    margin-bottom: 32px;
    
    .success-icon {
      font-size: 48px;
      color: #8E4EC6;
      margin-bottom: 16px;
      font-weight: bold;
    }
    
    h3 {
      margin: 0 0 8px 0;
      color: #2c3e50;
    }
    
    p {
      margin: 0;
      color: #666;
    }
  }
  
  .health-summary {
    margin-bottom: 32px;
    
    h4 {
      margin-bottom: 20px;
      color: #2c3e50;
    }
    
    .organ-health-grid {
      display: grid;
      gap: 16px;
      
      .organ-health-item {
        display: grid;
        grid-template-columns: 120px 1fr 50px;
        align-items: center;
        gap: 12px;
        
        .organ-name {
          font-weight: 500;
          color: #2c3e50;
        }
        
        .health-value {
          font-weight: 600;
          color: #2c3e50;
          text-align: right;
        }
      }
    }
  }
  
  .quiz-actions {
    display: flex;
    justify-content: center;
    gap: 12px;
  }
}

// Element Plus overrides for purple theme
:deep(.purple-button) {
  &.el-button--primary {
    background: linear-gradient(135deg, #8E4EC6, #9d4edd) !important;
    border-color: #8E4EC6 !important;
    
    &:hover {
      background: linear-gradient(135deg, #7b2cbf, #8E4EC6) !important;
      border-color: #7b2cbf !important;
    }
  }
  
  &.el-button--success {
    background: linear-gradient(135deg, #8E4EC6, #9d4edd) !important;
    border-color: #8E4EC6 !important;
    
    &:hover {
      background: linear-gradient(135deg, #7b2cbf, #8E4EC6) !important;
      border-color: #7b2cbf !important;
    }
  }
}

@media (max-width: 768px) {
  .health-quiz-container {
    padding: 12px;
  }
  
  .quiz-navigation {
    flex-direction: column;
    
    button {
      width: 100%;
    }
  }
  
  .quiz-actions {
    flex-direction: column;
    
    button {
      width: 100%;
    }
  }
}
</style>
