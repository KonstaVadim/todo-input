<template>
  <div id="app">
    <div class="todo-input-container" ref="todoInputContainer" @click="changeFocusOnInput()">
      <div class="displayed-input" ref="displayedInput"></div>
      <div class="fake-input" @focus="fieldFocus = true" @blur="fieldFocus = blurFromInput()" @input="changeText($event.target)" contenteditable="true" ref="fakeInput"></div>
      <div class="checkbox" @click.stop></div>
      <div class="calendar" @click.stop><i class="far fa-calendar-minus"></i></div>
      <div class="list" @click.stop>
        <span class="radio-btn"></span>
        <span>No list</span>
        <i class="arrow fas fa-angle-down"></i>
      </div>
      <div class="placeholder" v-show="!inputText">Write a new task</div>

      <transition name="fade">
      <div class="modal-placeholder" v-if="!fieldFocus && !inputText.length">
        <span>Write a new task</span>
      </div>
      </transition>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      clickTarget: null,
      inputText: '',
      fieldFocus: false,
      range: new Range(),
      addedComment: false,
      commentKey: '//',  // Ключ обозоначения комментария
    }
  },
  methods: {
    // Потеря фокуса с инпута, если кликнули вне его
    blurFromInput() {
      return this.$refs.todoInputContainer.contains(this.clickTarget);
    },
    // Сменить фокус на поле ввода
    changeFocusOnInput() {
      this.$refs.fakeInput.focus();
      this.fieldFocus = true;
    },
    // Изменяет высоту стилизованного инпута под спрятанный
    changeHeightBlock(target) {
      let height = target.offsetHeight;
      let displayedInput = this.$refs.displayedInput;
      if (height != displayedInput.offsetHeight) {
        displayedInput.style.height = height + 'px';
      }
    },
    // Метод для отслеживания изменения текста
    changeText(target) {
      let text = target.textContent;  // Введенный текст

      // Если ввели символ, обозначающий комментарий и еще не было выделения
      if (text.includes(this.commentKey) && !this.addedComment) {
        let indexCommentKey = text.indexOf(this.commentKey);  // Позиция, с которой комментарий начинается

        // Устанавливаем диапазон
        this.range.setStart(target.firstChild, indexCommentKey);
        this.range.setEnd(target.firstChild, text.length);

        const span = document.createElement('mark');  // Выделитель

        // Выделяем
        this.range.surroundContents(span);

        this.addedComment = true; // Флаг, что комментарий уже выделяется

        // Изменяем позицию курсора
        this.changeCursorPosition(window.getSelection().anchorNode.nextElementSibling, 1);
      } else if(!text.includes(this.commentKey) && this.addedComment) {  // Если комментарий стерли, убираем выделение
        target.innerHTML = text;  // Добавляем только текст

        let offset = text.length ? 1 : 0; // Дочерний элемент, к которому ставим курсор
        this.changeCursorPosition(target, offset);

        this.addedComment = false;  // Забываем о выделении
      }
      // Изменяем высоту
      this.changeHeightBlock(target);
      this.inputText = target.textContent;
    },
    // Изменяет расположение курсора
    changeCursorPosition(elem, offset) {
      this.range.setStart(elem, offset);
      window.getSelection().removeAllRanges();
      window.getSelection().addRange(this.range);
    }
  },
  created() {
    document.addEventListener('mousedown', event => {
      this.clickTarget = event.target;
    })
  },
  destroyed() {
    document.removeEventListener('mousedown', event => {
      this.clickTarget = event.target;
    })
  }
}
</script>

<style lang="scss">

@mixin input-style {
  padding: 17px 165px 17px 50px;
  outline: none;
  border-radius: 1rem;
  min-height: 59px;
  line-height: 25px;
  transition: all 0.3s cubic-bezier(0, 0.72, 1, 1.01);
}

@mixin df-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial;
}
body {
  background-color: #dee0e4;
}
#app {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.todo-input-container *{
  transition: all 0.3s ease;

}

.todo-input-container {
  position: relative;

  mark {
    color: #886f44;
    background-color: unset;
  }

  .fake-input {
    @include input-style;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    background-color: unset;

  }

  .displayed-input {
    @include input-style;
    position: relative;
    width: 450px;
    font-size: 1rem;
    color: #333431;
    background-color: #fff;
  }

  .checkbox {
    position: absolute;
    left: 15px;
    top: 18px;
    width: 24px;
    height: 24px;
    background-color: #eaeaea;
    border-radius: 6px;
  }

  .calendar {
    @include df-center;
    position: absolute;
    right: 120px;
    top: 15px;
    width: 30px;
    height: 30px;
    background-color: #f1f3f2;
    border-radius: 8px;
    color: #696d76;
  }

  .list {
    @include df-center;
    justify-content: space-between;
    position: absolute;
    right: 10px;
    top: 15px;
    width: 100px;
    height: 30px;
    padding: 0 10px;
    background-color: #f2f4f6;
    border-radius: 10px;
    color: #403e3b;

    .radio-btn {
      height: 10px;
      width: 10px;
      border-radius: 4px;
      border: 2px solid #c4c6c8;
    }

    .arrow {
      color: #696d76;
    }
  }

  .modal-placeholder {
    display: flex;
    align-items: center;
    padding-left: 20px;
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background-color: #ced2da;
    color: #7c7f85;
    border-radius: 1rem;
    cursor: pointer;

    &:hover {
      background-color: #c6c9d1;
    }
  }

  .placeholder {
    position: absolute;
    left: 50px;
    top: 21px;
    color: #ababab;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: all .3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
  transform: scale(0.3) translateX(-350px);
}

</style>
