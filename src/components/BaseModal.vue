<template>
  <!--это компонент для анимированного поля с ознакомлением приложение-->
  <!--тег ТЕЛЕПОРТ со свойством <body> который я изпользовал для модального окна 
  что бы оно смогло сводно перемещаться внутри всего приложения 
не беспокоясь о структуре DOM -->

  <Teleport to="body">
    <!--тег ТРАНЗИШН который позволяет добавить некоторые анимаций
    в модальное окно со специальными стилями для того что бы анимировать 
  модальное окно -->
    <Transition name="modal-outer">
      <div
        v-show="modalActive"
        class="absolute w-full bg-opacity-30 flex h-screen top-0 left-0 justify-center px-8"
      >
        <!--еще один транзишн для создания анимаций внутри 
        модального окна
      -->
        <Transition name="modal-inner">
          <div
            v-if="modalActive"
            class="p-4 bg-white self-start mt-32 max-w-screen-md"
          >
            <slot />
            <!--это кнопка "Close"
               и при клике на нее происходит событие close-modal
              собитие емит позволяющий компоненту бэйзмодал 
            использовать событие кнопки в других дочерних компонентах 
          через событие close-modal -->
            <button
              @click="$emit('close-modal')"
              class="text-white mt-8 bg-weather-primary py-2 px-6"
            >
              Close
            </button>
          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
//опция которая передает свойство с настройкам к дочерним компонентам благодаря которому можно
//слушать события с дочернего компонета из родительского
defineEmits(["close-modal"]);
// опция с вызовом функций к которому мы передаем обьект
// модалактив свойство в котором есть тайп булин что обозночает модалактив должен быть булевым значением
// и по умолчанию оно фолс значит модальное окно всегда закрыто если только не нажать на кнопку с событием
defineProps({
  modalActive: {
    type: Boolean,
    default: false,
  },
});
</script>

<!--специальные классы с которыми можно анимировать компонет из транзишн-->
<style scoped>
.modal-outer-enter-active,
.modal-outer-leave-active {
  transition: opacity 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-outer-enter-from,
.modal-outer-leave-to {
  opacity: 0;
}

.modal-inner-enter-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15;
}

.modal-inner-leave-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-inner-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.modal-inner-leave-to {
  transform: scale(0.8);
}
</style>
