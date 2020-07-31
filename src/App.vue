<template>
  <div v-cloak>
    <div>
      {{ message }}
    </div>
    <div class="mb-5">
      <input class="url" type="text" v-model="url" />
    </div>
    <transition name="slide" appear>
      <div class="mb-5">
        <input class="kataKunci" type="text" v-model="kataKunci" />
        <button class="bersihkan" @click="bersihkan" v-if="clear">
          Bersihkan
        </button>
      </div>
    </transition>
    <transition name="fade" mode="out-in">
      <div v-if="daftarHasil.length === 0 && kataKunci.length > 0">
        <p>
          No results for '<strong>{{ kataKunci }}</strong
          >'
        </p>
        <img src="https://unsplash.it/400?random" alt="" />
      </div>
    </transition>

    <div v-if="hasil">
      <div class="total mb-5">Total Hasil: {{ paginasi.total }}</div>
      <div class="halaman mb-5">Halaman ke: {{ paginasi.page }}</div>
    </div>

    <ul class="daftar">
      <transition-group name="slide">
        <li v-for="(hasil, index) in daftarHasil" :key="index" class="mb-3">
          <span v-html="hasil.text"></span>
          <a :href="`${url}&t=${hasil.start}s`" target="_blank">Youtube</a>
        </li>
      </transition-group>
    </ul>

    <transition name="fade">
      <div v-if="kataKunci.length < 1">
        <!-- <p>
          No results for '<strong>{{ kataKunci }}</strong
          >'
        </p> -->
        <img src="https://unsplash.it/200?random" alt="" />
      </div>
    </transition>
    <div class="con">
      <div class="pag">
        <button
          class="pertama"
          @click="navigasi('first')"
          :disabled="!paginasi['first']"
        >
          First
        </button>
        <button
          class="sebelumnya"
          @click="navigasi('prev')"
          :disabled="!paginasi['prev']"
        >
          Prev
        </button>
        <button
          class="selanjutnya"
          @click="navigasi('next')"
          :disabled="!paginasi['next']"
        >
          Next
        </button>
        <button
          class="terakhir"
          @click="navigasi('last')"
          :disabled="!paginasi['last']"
        >
          Last
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: "this is pre loader",
      url: "https://www.youtube.com/watch?v=klnvttPfOUM",
      kataKunci: "",
      daftarHasil: [],
      clear: false,
      hasil: false,
      paginasi: {
        first: null,
        last: null,
        prev: null,
        next: null,
        total: 0,
        page: null
      }
    };
  },
  watch: {
    kataKunci: async function tanganiKataKunci(kataKunci) {
      if (kataKunci && kataKunci.length >= 3) {
        await this.cari(kataKunci, this.url);
        this.hasil = true;
      } else {
        this.bersihkanHasilDanPaginasi();
      }
      if (this.kataKunci == "") {
        this.clear = false;
      } else {
        this.clear = true;
      }
    }
  },
  methods: {
    async cari(kataKunci, url, paginasi) {
      try {
        const respon = await fetch(
          paginasi
            ? paginasi
            : `https://cari-teks-video-api.vercel.app/api/search?q=${kataKunci}&url=${encodeURIComponent(
                url
              )}`
        ).then(_ => (_.ok ? _.json() : []));
        this.daftarHasil = respon.data;
        this.paginasi.first = respon.first;
        this.paginasi.last = respon.last;
        this.paginasi.prev = respon.prev;
        this.paginasi.next = respon.next;
        this.paginasi.total = respon.total;
        this.paginasi.page = respon.page;
      } catch (error) {}
    },
    async navigasi(type) {
      if (!this.paginasi[type]) {
        return;
      }
      await this.cari(this.kataKunci, this.url, this.paginasi[type]);
    },
    bersihkan() {
      this.kataKunci = "";
      this.bersihkanHasilDanPaginasi();
    },
    bersihkanHasilDanPaginasi() {
      this.daftarHasil = [];
      this.paginasi = {
        first: null,
        last: null,
        prev: null,
        next: null,
        total: 0,
        page: null
      };
    }
  }
};
</script>

<style>
[v-cloak] > * {
  display: none;
}
[v-cloak]::before {
  content: "candra";
  position: absolute;
  left: 50%;
  top: 50%;
  z-index: 1;
  width: 150px;
  height: 150px;
  margin: -75px 0 0 -75px;
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid #3498db;
  width: 120px;
  height: 120px;
  -webkit-animation: spin 2s linear infinite;
  animation: spin 2s linear infinite;
}
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
.fade-enter {
  opacity: 0;
}
.fade-enter-active {
  transition: opacity 0.8s;
}
.fade-leave {
}
.fade-leave-active {
  transition: opacity 0.8s;
  opacity: 0;
}

.slide-enter {
  opacity: 0;
}
.slide-enter-active {
  animation: slide-in 0.8s ease-out forwards;
  transition: opacity 0.8s;
}
.slide-leave {
  opacity: 0;
}
.slide-leave-active {
  animation: slide-out 0.8s ease-out forwards;
  transition: opacity 0.8s;
  opacity: 0;
  position: absolute;
}
.slide-move {
  transition: transform 0.8s;
}
@keyframes slide-in {
  from {
    transform: translateY(20px);
  }
  to {
    transform: translateY(0);
  }
}
@keyframes slide-out {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(20px);
  }
}
.con {
  position: relative;
}
.pag {
  top: 90vh;
  position: fixed;
}
.mb-5 {
  margin-bottom: 1rem;
}
.mb-3 {
  margin-bottom: 1rem;
}
</style>
