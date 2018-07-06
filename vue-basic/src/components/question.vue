<template>
  <div id="watch-example">
    <p>
      Задайте вопрос, на который можно ответить "да" или "нет":
      <input v-model="question">
      <input v-model="trigger">
    </p>
    <p>{{ answer }}</p>
  </div>
</template>

<script>

import axios from 'axios';
import lodash from 'lodash';

export default {
  data() {
    return {
      question: '',
      answer: 'Пока вы не зададите вопрос, я не могу ответить!',
      trigger: ''
    }
  },
  watch: {
    // эта функция запускается при любом изменении вопроса
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Ожидаю, когда вы закончите печатать...'
      this.debouncedGetAnswer()
    },
    trigger: function() {
      console.log(this);
    }
  },
  created: function () {
    // _.debounce — это функция из lodash, позволяющая ограничить
    // то, насколько часто может выполняться определённая операция.
    // В данном случае мы хотим ограничить частоту обращений к yesno.wtf/api,
    // дожидаясь завершения печати вопроса перед тем как послать ajax-запрос.
    // Чтобы узнать больше о функции _.debounce (и её родственнице _.throttle),
    // см. документацию: https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Вопросы обычно заканчиваются вопросительным знаком. ;-)'
        return
      }
      this.answer = 'Думаю...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Ошибка! Не могу связаться с API. ' + error
        })

    }
  }

}

</script>
