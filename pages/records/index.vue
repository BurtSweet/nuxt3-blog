<script setup lang="ts">
import { type RecordItem, getNowDayjs } from "~/utils/common";
import { formatTime, useListPage } from "~/utils/nuxt";

const recordList = await useListPage<RecordItem>();

const currentYear = useState<number | undefined>("record-expand", () => undefined);

const years = computed(() => {
  const result: {
    year: number;
    items: RecordItem[];
  }[] = [];
  recordList
    .forEach((item) => {
      const year = getNowDayjs(item.time).year();
      const exist = result.find(v => v.year === year);
      if (!exist) {
        result.push({
          year,
          items: [item]
        });
      } else {
        exist.items.push(item);
      }
    });
  return result.sort((a, b) => b.year - a.year);
});

onMounted(() => {
  if (years.value.length && !currentYear.value) {
    currentYear.value = years.value[0].year;
  }
});
</script>

<template>
  <div class="record-list">
    <ul v-if="years.length">
      <li v-for="year in years" v-show="year.items.some(item => item._show)" :key="year.year">
        <h2 class="flex" @click="currentYear = currentYear == year.year ? undefined : year.year">
          <svg-icon :style="{transform: currentYear==year.year ? 'rotate(180deg)' : ''}" name="up" />
          {{ year.year }}
          <span class="size">( {{ year.items.length }} {{ $t('items') }} )</span>
        </h2>
        <div v-show="currentYear==year.year" class="contain flex">
          <div v-for="item in year.items" v-show="item._show" :key="item.id" class="item flexc">
            <b :title="formatTime(item.time)">{{
              formatTime(item.time, 'month')
            }}</b>
            <nuxt-link
              :class="{ multiple: item.images.length > 1 }"
              no-prefetch
              :to="'/records/' + item.id"
            >
              <the-lazy-img
                v-if="currentYear==year.year"
                alt="cover"
                :src="item.images[0]?.src ?? 'no-poster'"
                :retry="false"
              />
            </nuxt-link>
          </div>
        </div>
      </li>
    </ul>
    <div v-else class="empty">
      {{ $T('nothing-here') }}
    </div>
  </div>
</template>

<style lang="scss">
$space: 16px;

.record-list {
  margin: 20px 30px 0 40px;
  padding-bottom: 60px;

  .common-loading {
    margin-top: calc(50vh - $header-height);
    transform: translateY(-100%);
  }

  .empty {
    color: rgb(53 53 53);
    font-size: 1rem;
    text-align: center;

    @include dark-mode {
      color: rgb(212 212 212);
    }

    margin-top: 50px;
  }

  ul {
    margin: auto;
    list-style: none;

    li {
      @include dark-mode {
        &::marker {
          color: white;
        }
      }

      h2 {
        margin: $space * 2 0;
        color: $theme-color-darken;
        text-shadow: 0 0 2px cyan;
        cursor: pointer;

        &:hover {
          text-decoration: underline;
        }

        svg {
          @include square(24px);

          transition: $common-transition;
          margin-right: 8px;
        }

        .size {
          font-size: f-size(0.86);
          color: #373737;
          margin-left: 8px;
          text-shadow: none;
        }

        @include dark-mode {
          color: $theme-color-lighten;
          text-shadow: 0 0 2px cyan;

          .size {
            color: #dedede;
          }
        }
      }

      &:hover {
        .contain {
          border-color: gray;
        }
      }

      .contain {
        flex-wrap: wrap;
        gap: $space * 2 0;

        .item {
          position: relative;
          margin: 0 $space * 1.8;

          &::before {
            position: absolute;
            content: "";
            width: calc(100% + #{$space * 3.6});
            height: 1px;
            background: #e9e9e9;
            top: 0;
          }

          &:not(:last-of-type) {
            &::after {
              position: absolute;
              content: "";
              transform: translateY(-50%);
              top: 0;
              right: -$space * 1.8;
              background: rgb(207 207 207);
              height: $space * 0.8;
              width: 1px;
            }
          }

          &:hover {
            b {
              color: $theme-color-darken;
              text-shadow: 0 0 2px cyan;

              @include dark-mode {
                color: $theme-color-lighten;
                background: $background-dark;
              }
            }
          }

          b {
            transform: translateY(-50%);
            padding: 0 5px;
            font-size: f-size(0.9);
            background: $background;

            @include dark-mode {
              background: $background-dark;
            }

            transition: $common-transition;
          }

          a {
            margin-top: 5px;
            position: relative;
            transition: $common-transition;

            @include square($space * 10);

            .--lazy-img {
              z-index: 3;

              span {
                border: 1px solid #dedede;
                box-sizing: border-box;
              }

              img {
                width: 100%;
                height: 100%;
                object-fit: cover;
              }
            }

            &:hover {
              .--lazy-img {
                transform: translate(-2px, -2px);
              }
            }

            &::before,
            &::after,
            .--lazy-img {
              position: absolute;
              top: 0;
              left: 0;

              @include square;

              transition: $common-transition;
            }

            &.multiple {
              &::before,
              &::after {
                content: "";
              }

              &::before {
                z-index: 2;
                transform: translate(4px, 4px);
                background: #b8b8b8;
              }

              &::after {
                z-index: 1;
                transform: translate(8px, 8px);
                background: #d8d8d8;
              }

              &:hover {
                &::after {
                  transform: translate(10px, 10px);
                }
              }
            }
          }
        }
      }
    }
  }
}

@include mobile {
  .record-list {
    margin: 20px 10px;

    ul li .contain {
      .item {
        &:nth-child(even) {
          &::after {
            content: none;
          }
        }

        a {
          @include square(calc(50vw - $space * 3.6 - 10px));
        }
      }
    }
  }
}
</style>
