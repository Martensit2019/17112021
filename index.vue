<template>
  <div class="kim">
    <div v-if="!startTest">
      <div class="kim__header mb_4">
        <div class="kim__header-left-section">
          <div class="kim__header-label mr_2">
            ТЕСТ {{ $route.params.kim_id }}
          </div>
          <span> {{ kim.data.course.name }} </span>
        </div>
      </div>
      <div class="kim-attention">
        Внимание!
        <p>
          Мы искренне надеемся, что Вы намерены пройти тестирование
          самостоятельно без чьей-либо помощи. Конечно, Вы можете использовать
          документацию, учебники и другую информацию в Сети.
        </p>
        <p>
          Не используйте кнопку браузера “Назад”, а также другие способы
          возврата к предыдущей странице.
        </p>
        <p>
          Оценка за пройденный этап выставляется в соответствии с количеством
          правильных ответов.
        </p>
        <p>Если Вы набрали:</p>
        <div class="rating-sheet">
          <div class="rating-sheet__item">
            <div class="rating-sheet__item--percent">&lt; 55%</div>
            <div class="rating-sheet__item--score">
              тестирование не пройдено
            </div>
            <div class="rating-sheet__item--num">2</div>
          </div>
          <div class="rating-sheet__item">
            <div class="rating-sheet__item--percent">&gt; 55 — 70%</div>
            <div class="rating-sheet__item--score">оценка</div>
            <div class="rating-sheet__item--num orange">3</div>
          </div>
          <div class="rating-sheet__item">
            <div class="rating-sheet__item--percent">&gt; 70 — 90%</div>
            <div class="rating-sheet__item--score">оценка</div>
            <div class="rating-sheet__item--num yellow">4</div>
          </div>
          <div class="rating-sheet__item">
            <div class="rating-sheet__item--percent">&gt; 90%</div>
            <div class="rating-sheet__item--score">оценка</div>
            <div class="rating-sheet__item--num green">5</div>
          </div>
        </div>
        <p>
          При повторной сдаче теста результаты предыдущей попытки аннулируются.
        </p>
        <p>
          После нажатия на кнопку “Подтверждаю” Вы приступите к сдаче теста.
        </p>
      </div>
      <div class="mt_4 mb_5">
        <input
          id="kim-agree"
          v-model="agree"
          class="profile__custom_input"
          type="checkbox"
        />
        <label class="profile__custom_label" for="kim-agree">
          <span class="profile_rch_title"
            >Я подтверждаю, что это задание я буду выполнять
            самостоятельно.</span
          >
        </label>
      </div>
      <div class="kim__control">
        <button
          class="btn non-focus kim_auto-width mr_2"
          :class="[agree ? 'btn__primary' : 'btn__gray']"
          @click="letsStarting"
        >
          Подтверждаю
        </button>
        <NuxtLink
          :to="{
            name: 'programs-program_id-course_id',
            params: {
              program_id: $route.params.program_id,
              course_id: $route.params.course_id,
            },
          }"
          class="btn btn__gray non-focus kim_auto-width"
        >
          Отменить
        </NuxtLink>
      </div>
    </div>
    <test
      v-else
      ref="test"
      :kim="kim.data.kim"
      :course="kim.data.course"
      @start-timer="retake()"
      @stop-timer="stopTimer()"
    />
  </div>
</template>

<script>
import Test from "~/components/programs/layouts/Test";

export default {
  name: "index",
  components: { Test },
  layout: "educationProgram",

  async asyncData({ $axios, params, redirect, error }) {
    const kim = await $axios
      .$get(
        `/programs/${params.program_id}/course/${params.course_id}/kim/${params.kim_id}`
      )
      .catch((invalid) => {
        if (invalid.response.status === 401) {
          redirect({ name: "login" });
        } else if (invalid.response.status === 404) {
          error({ statusCode: 404, message: "КИМ не найден" });
        }
      });
    return { kim };
  },
  data() {
    return {
      agree: false,
      startTest: false,
      interval: 0,
      intervalId: null,
    };
  },
  mounted() {
    this.$nextTick(() => {
      setTimeout(() => {
        this.$store.commit("categories/SET_LOADING", false);
      }, 500);
    });
    this.parseTime();
  },
  methods: {
    parseTime() {
      const days = this.kim.data.kim.duration.days
        ? this.kim.data.kim.duration.days * 86400
        : 0;
      const hours = this.kim.data.kim.duration.hours
        ? this.kim.data.kim.duration.hours * 3600
        : 0;
      const minutes = this.kim.data.kim.duration.minutes
        ? this.kim.data.kim.duration.minutes * 60
        : 0;
      this.interval = days + hours + minutes;
    },
    timer() {
      this.intervalId = setInterval(() => {
        if (this.interval === 0) {
          clearInterval(this.intervalId);
          this.$notify({
            group: "warning",
            text: "Время истекло",
          });
          this.$refs.test.stopTest();
          return;
        }
        this.interval -= 1;
      }, 1000);
    },
    letsStarting() {
      if (this.agree) {
        this.startTest = true;
        if (this.interval) {
          this.timer();
        }
      }
    },
    retake() {
      this.parseTime();
      this.timer();
    },
    stopTimer() {
      clearInterval(this.intervalId);
    },
  },
};
</script>

<style lang="scss" scoped>
.kim-attention {
  max-width: 870px;
  p {
    padding-top: 20px;
  }
}
.rating-sheet {
  max-width: 490px;
  &__item {
    display: flex;
    justify-content: space-between;
    height: 40px;
    align-items: center;
    border-bottom: 1px solid #f1f0f0;
  }
  &__item--percent {
  }
  &__item--score {
    margin-left: auto;
    margin-right: 10px;
  }
  &__item--num {
    width: 22px;
    height: 22px;
    line-height: 22px;
    border-radius: 50%;
    background-color: red;
    text-align: center;
    font-weight: bold;
    font-size: 16px;
    line-height: 24px;
    color: #ffffff;
  }
  .orange{
    background-color: #fd7e14;;
  }
  .yellow{
    background-color: #ffc107;
  } 
  .green{
    background-color: #40BF54;
  }
}
</style>
