<script>
import { format, parseISO } from 'date-fns';
export default {
  name: 'App',

  data() {
    return {
      loadedData: null,
      defaultSortedData: [],
      sortMode: 14, // 14 - сортировка по статусу, 28 - обратная сортировка по статусу
      noFinished: false, // не показывать завершенные
      noFutures: false // не показывать неначатые
    };
  },

  mounted() {
    this.loadData();
    this.sortedData = this.loadedData;
  },

  methods: {
    async loadData() {
      try {
        const response = await fetch('/data/data.json');
        const jsonData = await response.json();
        this.loadedData = jsonData;
        this.defaultSortedData = this.loadedData.map(item => ({ ...item, status: 0 }));
        
        const today = new Date();
        this.defaultSortedData.forEach(item => {
          const dates = item.dates;
          if (dates && dates.length > 0) {
            const start_min = new Date(Math.min(...dates.map(date => new Date(date.start_date))));
            const end_max = new Date(Math.max(...dates.map(date => new Date(date.end_date))));
            if (today <= start_min) {
              item.status = 10000 + Math.floor((start_min - today) / (1000 * 60 * 60 * 24));
              // Диета еще не стартовала
            } else if (today < end_max) {
              item.status = 0 - Math.floor((end_max - today) / (1000 * 60 * 60 * 24));
              // Диета в процесее
            } else if (today > end_max) {
              item.status = -50000 - Math.floor((today - end_max) / (1000 * 60 * 60 * 24));
              // Диета закончилась
            }
          }
        });
      } catch (error) {
        console.error('Error Loading JSON data:', error);
      }
    },
    
    formattedDate(date) {
      const parsedDate = parseISO(date);
      return format(parsedDate, 'dd MMM');
    },

    changeSortMode() {
      if (this.sortMode <= 14) {
        this.sortMode += 14;
      } else if (this.sortMode > 14) {
        this.sortMode -= 14;
      }
    }
  },

  computed: {
    sortedData() {
      if (this.sortMode === 14) {
        return this.defaultSortedData.slice().sort((a, b) => b.status - a.status);
      } else if (this.sortMode === 28) {
        return this.defaultSortedData.slice().sort((a, b) => a.status - b.status);
      } else {
        return this.defaultSortedData.slice();
      }
    },

    filteredSortedData() {
      const filteredData = this.sortedData.filter(item => {
        if (this.noFinished && item.status <= -50000) {
          return false;
        }
        if (this.noFutures && item.status >= 10000) {
          return false;
        }
        return true;
      });
      return filteredData;
    }
  }
}
</script>

<template>
  <div class="container">
    <div class="row">
      <div class="col-8 text-center" ><h3>Название таблицы</h3></div>
        
      <div class="col-1"></div>
      <div class="col-3">
      <div class="form-check">
        <input class="form-check-input" type="checkbox" value="" v-model="noFinished" id="checkbox-1">
          <label class="form-check-label" for="flexCheckDefault">
            Не показывать завершенные
          </label>
        </div>
        <div class="form-check">
        <input class="form-check-input" type="checkbox" value="" v-model="noFutures" id="checkbox-2">
          <label class="form-check-label" for="flexCheckDefault">
            Не показывать неначатые
          </label>
        </div>
      </div>
    </div>
  </div>

  <table class="table table-sm table-bordered table-striped align-middle">
    <thead>
      <tr class="">
        <th scope="col" class="align-middle text-center">id</th>
        <th scope="col" class="align-middle text-center">Клиент</th>
        <th scope="col" class="align-middle text-center">Диета</th>
        <th scope="col" class="align-middle text-center">Тариф</th>
        <th scope="col" class="align-middle text-center">Адрес</th>
        <th scope="col" class="align-middle text-center">Телефон</th>
        <th scope="col" class="align-middle text-center">Даты</th>
        <th scope="col" class="align-middle text-center">Скидка</th>
        <th scope="col" class="align-middle text-center">Заказов сделано</th>
        <th scope="col" class="align-middle text-center">Заказов оплачено</th>
        <th scope="col" class="align-middle text-center">Статус оплаты</th>
        <th scope="col" class="align-middle text-center">Комментарий курьера</th>
        <th scope="col" class="align-middle text-center">Внутренний комментарий</th>
        <th @click="changeSortMode" scope="col" class="align-middle text-center" style="cursor: pointer;" title="Реверсировать сортировку">Статус</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(client) in filteredSortedData" :key="client.o_id">
        <th scope="row">{{ client.o_id }}</th>
        <td>{{ client.client_name }}</td>
        <td class="text-center">
          <div v-for="(diet, index) in client.diets" :key="diet">
            {{ diet }}
            <hr v-if="index < client.diets.length - 1">
          </div>
        </td>
        <td class="text-center">
          <div v-for="(tariff, index) in client.tariff" :key="tariff">
            {{ tariff }}
            <hr v-if="index < client.tariff.length - 1">
          </div>
        </td>
        <td class="text-center">{{ client.address }}</td>
        <td class="text-center">{{ client.phone }}</td>
        <td class="text-center">
          <div v-for="(dates, index) in client.dates" :key="dates">
            <span style="white-space: nowrap">{{ formattedDate(dates.start_date) }} - {{ formattedDate(dates.end_date) }}</span>
            <hr v-if="index < client.dates.length - 1">
          </div>        
        </td>
        <td class="text-center">{{ client.discount }}</td>
        <td class="text-center">{{ client.order_sum }}</td>
        <td class="text-center">{{ client.order_payed }}</td>
        <td class="text-center">{{ client.pay_status }}</td>
        <td class="text-center">{{ client.courier_comment }}</td>
        <td class="text-center">{{ client.inner_comment }}</td>
        <td v-if="client.status >= 10000" class="text-center">Дней до начала: {{ client.status - 10000 }}</td>
        <td v-if="client.status == 0" class="text-center">Заканчивается сегодня</td>
        <td v-if="client.status == -1" class="text-center">Заканчивается завтра</td>
        <td v-if="client.status < 10000 && client.status != -1 && client.status != 0 && client.status > -50000" class="text-center">Дней до окончания: {{ client.status * -1 }}</td>
        <td v-if="client.status <= -50000" class="text-center">Закончилось {{ (client.status + 50000) * -1 }} дней назад</td>
      </tr>
    </tbody>
  </table>  
</template>

<style>

</style>
