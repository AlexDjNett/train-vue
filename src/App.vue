<template>
  <div class="container mx-auto flex flex-col items-center p-4">
    <div
      v-if="isLoading"
      class="w-100 h-100 fixed inset-0 z-50 flex items-center justify-center bg-purple-800 opacity-80"
    >
      <svg
        class="-ml-1 mr-3 h-12 w-12 animate-spin text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div>
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"
              >Тикер {{ ticker }}</label
            >
            <div class="relative mt-1 rounded-md shadow-md">
              <input
                @keydown.enter="add"
                v-model="ticker"
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full rounded-md border-gray-300 pr-10 text-gray-900 focus:border-gray-500 focus:outline-none focus:ring-gray-500 sm:text-sm"
                placeholder="Например DOGE"
              />
            </div>
            <div
              v-if="filteredCoins.length && ticker !== ''"
              class="flex flex-wrap rounded-md bg-white p-1 shadow-md"
            >
              <span
                class="m-1 inline-flex cursor-pointer items-center rounded-md bg-gray-300 px-2 text-xs font-medium text-gray-800"
                v-for="coin in filteredCoins"
                :key="coin.Symbol"
                @click="add(coin.Symbol)"
              >
                {{ coin.Symbol }}
              </span>
            </div>
            <div class="text-sm text-red-600" v-if="coinError">Такой тикер уже добавлен</div>
          </div>
        </div>
        <button
          @click="add"
          type="button"
          class="my-4 inline-flex items-center rounded-full border border-transparent bg-gray-600 px-4 py-2 text-sm font-medium leading-4 text-white shadow-sm transition-colors duration-300 hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
        >
          <!-- Heroicon name: solid/mail -->
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            xmlns="http://www.w3.org/2000/svg"
            width="30"
            height="30"
            viewBox="0 0 24 24"
            fill="#ffffff"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>

      <template v-if="tickers.length">
        <p class="flex items-center gap-x-4">
          <button
            v-if="page > 1"
            @click="page = page - 1"
            class="my-4 inline-flex items-center rounded-full border border-transparent bg-gray-600 px-4 py-2 text-sm font-medium leading-4 text-white shadow-sm transition-colors duration-300 hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
          >
            Назад</button
          ><button
            @click="page = page + 1"
            v-if="hasNextPage"
            class="my-4 inline-flex items-center rounded-full border border-transparent bg-gray-600 px-4 py-2 text-sm font-medium leading-4 text-white shadow-sm transition-colors duration-300 hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
          >
            Вперед
          </button>
          <span>Фильтр:</span> <input type="text" v-model="filter" />
        </p>
        <hr class="my-4 w-full border-t border-gray-600" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            class="cursor-pointer overflow-hidden rounded-lg border-solid border-purple-800 bg-white shadow"
            v-for="t in filteredTickers()"
            :class="{ 'border-4': sel === t }"
            @click="select(t)"
            :key="t.name"
          >
            <div class="px-4 py-5 text-center sm:p-6">
              <dt class="truncate text-sm font-medium text-gray-500">{{ t.name }} - USD</dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">{{ t.price }}</dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="handleDelete(t)"
              class="text-md flex w-full items-center justify-center bg-gray-100 px-4 py-4 font-medium text-gray-500 transition-all hover:bg-gray-200 hover:text-gray-600 hover:opacity-20 focus:outline-none sm:px-6"
            >
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="my-4 w-full border-t border-gray-600" />
      </template>
      <section class="relative" v-if="sel">
        <h3 class="my-8 text-lg font-medium leading-6 text-gray-900">{{ sel.name }} - USD</h3>
        <div class="flex h-64 items-end border-b border-l border-gray-600">
          <div
            :style="{ height: `${bar}%` }"
            :key="idx"
            v-for="(bar, idx) in normalizeGraph()"
            class="w-10 border bg-purple-800"
          ></div>
        </div>
        <button @click="sel = null" type="button" class="absolute right-0 top-0">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background: new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'App',
    data() {
      return {
        ticker: '',
        isLoading: true,
        tickers: [],
        sel: null,
        graph: [],
        coinList: [],
        coinError: false,
        page: 1,
        filter: '',
        hasNextPage: true
      };
    },

    async mounted() {
      this.$nextTick(function () {
        this.isLoading = false;
      });
      const t = await fetch(
        `https://min-api.cryptocompare.com/data/all/coinlist?summary=true&api_key=d0e5e594fd8062c73043dfaeaa36f179dae5bdaf75ece1020b6458f68072ef54`
      );

      const data = await t.json();

      this.coinList = Object.values(data.Data);
    },
    computed: {
      filteredCoins() {
        return this.coinList
          .filter((coin) => coin.Symbol.toLowerCase().includes(this.ticker.toLowerCase()))
          .slice(0, 4);
      }
    },
    watch: {
      ticker() {
        this.coinError = false;
      },
      filter() {
        this.page = 1;
        window.history.pushState(
          null,
          document.title,
          `${window.location.pathname}?filter=${this.filter}&page=${this.page}`
        );
      },
      page() {
        window.history.pushState(
          null,
          document.title,
          `${window.location.pathname}?filter=${this.filter}&page=${this.page}`
        );
      }
    },
    created() {
      const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());
      if (windowData.page) {
        this.page = windowData.page;
      }
      if (windowData.filter) {
        this.filter = windowData.filter;
      }
      const tickersData = localStorage.getItem('cryptonomicon-list');
      if (tickersData) {
        this.tickers = JSON.parse(tickersData);
        this.tickers.forEach(({ name }) => {
          this.subscribeToUpdates(name);
        });
      }
    },
    methods: {
      filteredTickers() {
        const start = (this.page - 1) * 6;
        const end = this.page * 6;

        const filteredTickers = this.tickers.filter((ticker) => ticker.name.includes(this.filter));
        this.hasNextPage = filteredTickers.length > end;
        return filteredTickers.slice(start, end);
      },
      subscribeToUpdates(tickerName) {
        setInterval(async () => {
          const f = await fetch(
            `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=${import.meta.env.API_KEY}`
          );
          const data = await f.json();
          this.tickers.find((t) => t.name === tickerName).price =
            data.USD > 1 ? data.USD.toFixed(1) : data.USD.toPrecision(2);
          if (this.sel?.name === tickerName) {
            this.graph.push(data.USD);
            console.log(data.USD);
          }
        }, 5000);

        this.ticker = '';
      },
      add(coin) {
        if (coin instanceof PointerEvent) {
          coin = undefined;
        }

        const ticker = coin ?? this.ticker;

        if (this.ticker === '') return;
        this.coinError = false;

        if (this.tickers.find((t) => t.name === ticker)) {
          this.coinError = true;
          return;
        }
        const currentTicker = {
          name: ticker,
          price: '-'
        };

        this.tickers.push(currentTicker);
        this.filter = '';

        localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));

        this.subscribeToUpdates(currentTicker.name);
      },
      handleDelete(tickerToRemove) {
        this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
      },

      normalizeGraph() {
        const maxValue = Math.max(...this.graph);
        const minValue = Math.min(...this.graph);
        return this.graph.map((price) => 5 + ((price - minValue) * 95) / (maxValue - minValue));
      },

      select(ticker) {
        this.sel = ticker;
        this.graph = [];
      }
    }
  };
</script>
