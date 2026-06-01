<script setup>
import { ref, computed, onMounted } from 'vue'

// --- STATE MANAGEMENT ---
const halamanSekarang = ref('home') 
const asal = ref('')
const tujuan = ref('')
const tanggal = ref('')
const jumlahKursi = ref(1)
const kelas = ref('')

const busPilihan = ref(null) 
const susunanKursi = ref([]) 
const kursiYangDipilih = ref([]) 
const noHp = ref('')

// DATA PENUMPANG DINAMIS (Array)
const dataPenumpang = ref([])

// STATE BUAT PEMBAYARAN & TIKET
const titikNaik = ref('')
const titikTurun = ref('')
const metodePembayaran = ref('')
const bookingCode = ref('') 
const menitSisa = ref(30) 
const detikSisa = ref(0)
let timerInterval = null

// --- FITUR KODE PROMO ---
const userInputPromoCode = ref('')
const discount = ref(0) 
const promoMessage = ref('')
const promoStatus = ref('idle') 

const validPromoCodes = {
  'KCPXBCA': 40000,
  'LIBURANA6': 60000
}

const terapkanPromoCode = () => {
  promoMessage.value = ''
  promoStatus.value = 'idle'
  discount.value = 0

  const kodeInput = userInputPromoCode.value.trim().toUpperCase()
  const potongan = validPromoCodes[kodeInput]

  if (potongan) {
    discount.value = potongan
    promoStatus.value = 'applied'
    promoMessage.value = `Diskon ${formatRupiah(potongan)} berhasil dipakai Sob! 🔥`
  } else {
    discount.value = 0
    promoStatus.value = 'invalid'
    promoMessage.value = 'Kode salah atau sudah expired Sob!'
  }
}

// --- FITUR CEK TIKET ---
const inputCekKode = ref('')
const inputCekWa = ref('')

const prosesCekTiket = () => {
  if (!inputCekKode.value || !inputCekWa.value) {
    alert('Isi Kode Booking dan Nomor WhatsApp dulu ya Sob!')
    return
  }
  
  // Karena nggak pake database, kita cek sama data yang baru dibooking di sesi ini
  if (bookingCode.value !== '' && inputCekKode.value.toUpperCase() === bookingCode.value && inputCekWa.value === noHp.value) {
    halamanSekarang.value = 'lihat-e-tiket'
    inputCekKode.value = ''
    inputCekWa.value = ''
  } else {
    alert('Waduh, tiket lo nggak ketemu nih. Pastiin kodenya bener atau coba pesen tiket dulu!')
  }
}

// --- DATABASE TITIK NAIK & TURUN ---
const daftarTitik = {
  'Jakarta': ['Terminal Pulo Gebang', 'Agen Pondok Pinang', 'Agen Lebak Bulus', 'Terminal Kp. Rambutan', 'Terminal Grogol', 'Pasar Minggu', 'Pool Kecapi Trans Kebayoran Baru'],
  'Bandung': ['Terminal Cicaheum', 'Agen Pasteur', 'Pool Kecapi Trans Suci', 'Agen Cileunyi'],
  'Semarang': ['Pool Krapyak', 'Agen Banyumanik', 'Agen Ungaran'],
  'Jepara': ['Terminal Jepara', 'Agen Pecangaan', 'Agen Bangsri'],
  'Malang': ['Terminal Arjosari', 'Pool P. Sudirman', 'Agen Klojen', 'Agen Singosari', 'Agen Lawang'],
  'Surabaya': ['Redbus Lounge'],
  'Cirebon': ['Pool Ciperna'],
  'Yogyakarta': ['Terminal Jombor', 'Terminal Giwangan']
}

// --- SISTEM SLIDER GAMBAR ---
const daftarGambar = ref([
  '/kcp_skylander.png',
  '/kcp_biru.jpg',
  '/cabin.png'
])
const indexGambar = ref(0)

onMounted(() => {
  setInterval(() => {
    indexGambar.value = (indexGambar.value + 1) % daftarGambar.value.length
  }, 4000) 
})

// --- PABRIK DATA JADWAL & KELAS BUS ---
const jadwalBus = ref([])

const ruteDasar = [
  { asal: 'Jakarta', tujuan: 'Malang', hargaBase: 300000 },
  { asal: 'Jakarta', tujuan: 'Yogyakarta', hargaBase: 220000 },
  { asal: 'Jakarta', tujuan: 'Surabaya', hargaBase: 280000 },
  { asal: 'Jakarta', tujuan: 'Semarang', hargaBase: 190000 },
  { asal: 'Jakarta', tujuan: 'Jepara', hargaBase: 210000 }
]

const cetakanKelas = [
  { 
    kelas: 'Executive', konfigurasi: '2-2', totalSeat: 30, 
    waktuBerangkat: '08:00', waktuTiba: '18:00', pengaliHarga: 1.0, 
    bodi: 'Skylander R25', sasis: 'Hino RM 280', 
    fitur: ['AC', 'Toilet', 'Reclining Seat', 'Leg Rest', 'Snack Box', 'Bantal & Selimut'] 
  },
  { 
    kelas: 'Super Executive', konfigurasi: '2-1', totalSeat: 21, 
    waktuBerangkat: '16:00', waktuTiba: '02:00', pengaliHarga: 1.3, 
    bodi: 'Jetbus 5 SHD', sasis: 'Scania K410', 
    fitur: ['AC', 'Toilet', 'Kursi Lebar 2-1', 'Leg Rest Ekstra', 'Makan Prasmanan', 'AVOD', 'Smoking Area'] 
  },
  { 
    kelas: 'Sleeper', konfigurasi: '1-1', totalSeat: 22, 
    waktuBerangkat: '19:00', waktuTiba: '05:00', pengaliHarga: 1.8, 
    bodi: 'Skylander R25 Sleeper', sasis: 'MAN RR4 26.480',
    fitur: ['Private Cabin', 'AC', 'Toilet', 'Kasur Rebah Full', 'Makan Premium', 'AVOD Android', 'Sandal Hotel'] 
  }
]

let idBus = 1
ruteDasar.forEach(rute => {
  cetakanKelas.forEach(cetakan => {
    jadwalBus.value.push({
      id: idBus,
      nomorLambung: `KCP-${100 + idBus}`,
      asal: rute.asal,
      tujuan: rute.tujuan,
      kelas: cetakan.kelas,
      konfigurasi: cetakan.konfigurasi,
      totalSeat: cetakan.totalSeat,
      sisaSeat: cetakan.totalSeat - Math.floor(Math.random() * (cetakan.totalSeat / 2)),
      harga: rute.hargaBase * cetakan.pengaliHarga,
      bodi: cetakan.bodi,
      sasis: cetakan.sasis,
      waktuBerangkat: cetakan.waktuBerangkat,
      waktuTiba: cetakan.waktuTiba,
      fitur: cetakan.fitur
    })
    idBus++
  })
})

const jadwalTersaring = computed(() => {
  return jadwalBus.value.filter((bus) => {
    const ruteCocok = bus.asal === asal.value && bus.tujuan === tujuan.value
    const kelasCocok = kelas.value === '' || bus.kelas === kelas.value
    return ruteCocok && kelasCocok
  })
})

const cariJadwal = () => {
  if (!asal.value || !tujuan.value || !tanggal.value) {
    alert('Isi Kota Asal, Tujuan, dan Tanggal dulu ya Sob!')
    return
  }
  halamanSekarang.value = 'hasil-pencarian'
}

const gantiHalaman = (halamanBaru) => {
  halamanSekarang.value = halamanBaru
  // Reset window ke atas pas pindah halaman
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const formatRupiah = (angka) => {
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', maximumFractionDigits: 0 }).format(angka)
}

const lihatDetail = (bus) => {
  alert(`Detail Armada Kecapi Trans:\n\nNomor: ${bus.nomorLambung}\nKelas: ${bus.kelas}\nBodi: ${bus.bodi}\nSasis: ${bus.sasis}\nKonfigurasi: ${bus.konfigurasi} (${bus.totalSeat} Seat)\n\nFasilitas:\n- ${bus.fitur.join('\n- ')}`)
}

const bikinDenah = (bus) => {
  susunanKursi.value = []
  let nomor = 1
  let pola = []

  if (bus.konfigurasi === '2-2') pola = ['kursi', 'kursi', 'lorong', 'kursi', 'kursi']
  else if (bus.konfigurasi === '2-1') pola = ['kursi', 'kursi', 'lorong', 'kursi']
  else if (bus.konfigurasi === '1-1') pola = ['kursi', 'lorong', 'kursi']

  while (nomor <= bus.totalSeat) {
    for (let j = 0; j < pola.length; j++) {
      if (pola[j] === 'kursi') {
        if (nomor <= bus.totalSeat) {
          susunanKursi.value.push({ id: nomor, tipe: 'kursi' })
          nomor++
        } else {
          susunanKursi.value.push({ tipe: 'kosong' })
        }
      } else {
        susunanKursi.value.push({ tipe: 'lorong' })
      }
    }
  }
}

const lanjutPilihKursi = (bus) => {
  busPilihan.value = bus 
  kursiYangDipilih.value = [] 
  bikinDenah(bus) 
  halamanSekarang.value = 'pilih-kursi' 
}

const pilihKursiIni = (kursi) => {
  if (kursi.tipe !== 'kursi') return

  const index = kursiYangDipilih.value.indexOf(kursi.id)
  if (index > -1) {
    kursiYangDipilih.value.splice(index, 1)
  } else {
    if (kursiYangDipilih.value.length < parseInt(jumlahKursi.value)) {
      kursiYangDipilih.value.push(kursi.id)
    } else {
      alert(`Woy Sob, lo di awal pesennya cuma ${jumlahKursi.value} kursi doang!`)
    }
  }
}

const bukaPembayaran = () => {
  titikNaik.value = ''
  titikTurun.value = ''
  metodePembayaran.value = ''
  userInputPromoCode.value = ''
  discount.value = 0
  promoStatus.value = 'idle'
  promoMessage.value = ''

  dataPenumpang.value = []
  for(let i = 0; i < parseInt(jumlahKursi.value); i++) {
    dataPenumpang.value.push({ nama: '', makan: '' })
  }
  
  halamanSekarang.value = 'pilih-metode-pembayaran'
}

const subtotal = computed(() => {
  if (!busPilihan.value) return 0 
  return kursiYangDipilih.value.length * busPilihan.value.harga
})

const adminFee = 15000 

const totalAkhir = computed(() => {
  if (subtotal.value === 0) return 0 
  return subtotal.value + adminFee - discount.value
})

const lanjutBayarDetail = () => {
  const cekPenumpangLengkap = dataPenumpang.value.every(p => p.nama.trim() !== '' && p.makan !== '')
  
  if (!cekPenumpangLengkap || !noHp.value || !titikNaik.value || !titikTurun.value || !metodePembayaran.value) {
    alert('Waduh Sob, isi data tiap penumpang, servis makan, titik naik/turun, dan metode pembayaran dulu!')
    return
  }
  menitSisa.value = 30; 
  detikSisa.value = 0;
  mulaiTimer();
  halamanSekarang.value = 'menunggu-pembayaran'
}

const konfirmasiBayar = () => {
  if (timerInterval) clearInterval(timerInterval); 
  bookingCode.value = 'KCP' + Math.random().toString(36).substring(2, 9).toUpperCase();
  halamanSekarang.value = 'pembayaran-sukses'
}

const lihatETiket = () => {
  halamanSekarang.value = 'lihat-e-tiket'
}

const mulaiTimer = () => {
  if (timerInterval) clearInterval(timerInterval); 
  timerInterval = setInterval(() => {
    if (detikSisa.value === 0) {
      if (menitSisa.value === 0) {
        clearInterval(timerInterval);
        alert('Waktu pembayaran lo abis! Pesenan kursi lo ke-cancel otomatis.')
        gantiHalaman('home')
        return;
      }
      menitSisa.value--;
      detikSisa.value = 59;
    } else {
      detikSisa.value--;
    }
  }, 1000); 
}
</script>

<template>
  <div class="min-h-screen bg-slate-950 text-slate-200 font-sans selection:bg-yellow-400 selection:text-blue-950">
    
    <!-- NAVBAR (Glassmorphism + Yellow Accent) -->
    <nav class="bg-slate-950/80 backdrop-blur-lg border-b border-blue-900/40 p-4 sticky top-0 z-50">
      <div class="max-w-6xl mx-auto flex justify-between items-center">
        <h1 class="text-2xl font-black text-white italic tracking-widest cursor-pointer hover:scale-105 transition transform" @click="gantiHalaman('home')">
          KECAPI <span class="text-yellow-400">TRANS</span>
        </h1>
        <div class="text-slate-300 font-semibold hidden md:flex space-x-8">
          <button @click="gantiHalaman('home')" class="hover:text-yellow-400 transition-colors duration-300" :class="halamanSekarang === 'home' ? 'text-yellow-400' : ''">Beranda</button>
          <button @click="gantiHalaman('cek-tiket')" class="hover:text-yellow-400 transition-colors duration-300" :class="halamanSekarang === 'cek-tiket' ? 'text-yellow-400' : ''">Cek Tiket</button>
          <button @click="gantiHalaman('info-armada')" class="hover:text-yellow-400 transition-colors duration-300" :class="halamanSekarang === 'info-armada' ? 'text-yellow-400' : ''">Info Armada</button>
        </div>
      </div>
    </nav>

    <!-- WRAPPER ANIMASI VUE (Smooth Transitions) -->
    <Transition name="fade" mode="out-in">
      
      <!-- ================= 1. HALAMAN HOME ================= -->
      <div v-if="halamanSekarang === 'home'" key="home">
        <div class="relative bg-cover bg-center h-[500px] flex items-center justify-center overflow-hidden transition-all duration-1000 ease-in-out"
             :style="{ backgroundImage: `url(${daftarGambar[indexGambar]})` }">
          <div class="absolute inset-0 bg-gradient-to-b from-slate-950/40 via-slate-950/70 to-slate-950"></div>
          <div class="z-10 text-center px-4 -mt-16">
            <h2 class="text-4xl md:text-6xl font-extrabold text-white mb-4 tracking-tight drop-shadow-2xl">
              Level Baru <span class="text-yellow-400">Perjalanan</span>
            </h2>
            <p class="text-lg md:text-xl text-slate-300 font-medium max-w-2xl mx-auto drop-shadow-[0_1.2px_1.2px_rgba(0,0,0,0.8)]">
              Rasakan sensasi armada premium dengan privasi kelas atas. Lebih dari sekadar tiket, ini adalah pengalaman.
            </p>
          </div>
        </div>

        <div class="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8 -mt-32 relative z-20 pb-20">
          <div class="bg-blue-950/40 backdrop-blur-xl rounded-[2rem] shadow-[0_0_40px_rgba(0,0,0,0.5)] border border-blue-900/50 p-8 md:p-10">
            <form @submit.prevent="cariJadwal" class="grid grid-cols-1 md:grid-cols-3 gap-6 items-end">
              <div>
                <label class="block text-sm font-bold text-blue-200/70 mb-2 tracking-wide uppercase">Kota Asal</label>
                <select v-model="asal" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 focus:border-yellow-400 outline-none transition text-white font-medium bg-slate-950/60 appearance-none">
                  <option value="" disabled selected>Pilih Keberangkatan</option>
                  <option v-for="kota in Object.keys(daftarTitik)" :key="kota" :value="kota">{{ kota }}</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-bold text-blue-200/70 mb-2 tracking-wide uppercase">Kota Tujuan</label>
                <select v-model="tujuan" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 focus:border-yellow-400 outline-none transition text-white font-medium bg-slate-950/60 appearance-none">
                  <option value="" disabled selected>Pilih Destinasi</option>
                  <option v-for="kota in Object.keys(daftarTitik)" :key="kota" :value="kota">{{ kota }}</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-bold text-blue-200/70 mb-2 tracking-wide uppercase">Tanggal</label>
                <input type="date" v-model="tanggal" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 focus:border-yellow-400 outline-none transition text-white font-medium bg-slate-950/60 [color-scheme:dark]" />
              </div>
              <div>
                <label class="block text-sm font-bold text-blue-200/70 mb-2 tracking-wide uppercase">Penumpang</label>
                <select v-model="jumlahKursi" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 focus:border-yellow-400 outline-none transition text-white font-medium bg-slate-950/60 appearance-none">
                  <option value="1">1 Penumpang</option>
                  <option value="2">2 Penumpang</option>
                  <option value="3">3 Penumpang</option>
                  <option value="4">4 Penumpang</option>
                </select>
              </div>
              <div>
                <label class="block text-sm font-bold text-blue-200/70 mb-2 tracking-wide uppercase">Kelas</label>
                <select v-model="kelas" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 focus:border-yellow-400 outline-none transition text-white font-medium bg-slate-950/60 appearance-none">
                  <option value="">Semua Kelas</option>
                  <option value="Executive">Executive</option>
                  <option value="Super Executive">Super Executive</option>
                  <option value="Sleeper">Sleeper</option>
                </select>
              </div>
              <div>
                <button type="submit" class="w-full bg-gradient-to-r from-yellow-500 to-yellow-400 hover:from-yellow-400 hover:to-yellow-300 text-blue-950 font-black text-lg p-4 rounded-2xl shadow-[0_0_20px_rgba(250,204,21,0.3)] transition duration-300 transform hover:-translate-y-1">
                  CARI TIKET
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>

      <!-- ================= HALAMAN BARU: CEK TIKET ================= -->
      <div v-else-if="halamanSekarang === 'cek-tiket'" key="cektiket" class="max-w-2xl mx-auto px-4 py-20 min-h-[70vh] flex flex-col justify-center">
        <div class="bg-blue-950/40 rounded-3xl border border-blue-900/50 p-10 shadow-2xl relative overflow-hidden text-center">
          <div class="absolute top-0 right-0 w-32 h-32 bg-yellow-400/10 blur-[60px] rounded-full pointer-events-none"></div>
          
          <div class="text-6xl mb-6">🎟️</div>
          <h2 class="text-3xl font-black text-white mb-2">Cek Status Tiket Lo</h2>
          <p class="text-slate-400 font-medium mb-10">Masukin Kode Booking sama Nomor WhatsApp yang lo pake pas mesen.</p>
          
          <form @submit.prevent="prosesCekTiket" class="space-y-6 text-left">
            <div>
              <label class="block text-sm font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Kode Booking (KCPxxxxx)</label>
              <input type="text" v-model="inputCekKode" placeholder="Contoh: KCPXYZ123" class="w-full border border-blue-900/60 rounded-xl p-4 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white font-bold tracking-widest uppercase bg-slate-950/50 placeholder-slate-600" />
            </div>
            <div>
              <label class="block text-sm font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Nomor WhatsApp</label>
              <input type="number" v-model="inputCekWa" placeholder="0812xxxxxxx" class="w-full border border-blue-900/60 rounded-xl p-4 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white font-medium bg-slate-950/50 placeholder-slate-600" />
            </div>
            <button type="submit" class="w-full mt-4 bg-gradient-to-r from-yellow-500 to-yellow-400 hover:from-yellow-400 hover:to-yellow-300 text-blue-950 font-black text-lg p-4 rounded-xl shadow-[0_0_20px_rgba(250,204,21,0.3)] transition duration-300 transform hover:-translate-y-1">
              CEK E-TIKET
            </button>
          </form>
        </div>
      </div>

      <!-- ================= HALAMAN BARU: INFO ARMADA ================= -->
      <div v-else-if="halamanSekarang === 'info-armada'" key="infoarmada" class="max-w-6xl mx-auto px-4 py-16">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-black text-white mb-4">Armada <span class="text-yellow-400">Sultan</span></h2>
          <p class="text-slate-400 font-medium max-w-2xl mx-auto text-lg">
            Intip kelas armada Kecapi Trans yang siap manjain perjalanan lo. Dari kelas Executive sampai private Sleeper, kita punya semuanya.
          </p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div v-for="(kelasData, index) in cetakanKelas" :key="index" class="bg-blue-950/40 rounded-[2.5rem] border border-blue-900/50 overflow-hidden shadow-2xl hover:-translate-y-2 transition transform duration-300 group">
            <!-- Pura-puranya ambil dari daftarGambar sesuai index -->
            <div class="h-56 bg-cover bg-center border-b border-blue-900/50 relative overflow-hidden" :style="{ backgroundImage: `url(${daftarGambar[index % daftarGambar.length]})` }">
              <div class="absolute inset-0 bg-gradient-to-t from-slate-950 to-transparent"></div>
              <div class="absolute bottom-4 left-6">
                <span class="bg-yellow-400/20 text-yellow-400 border border-yellow-400/30 text-xs tracking-widest uppercase font-black px-3 py-1 rounded-full backdrop-blur-sm">
                  {{ kelasData.konfigurasi }} Seat
                </span>
              </div>
            </div>
            
            <div class="p-8">
              <h3 class="text-2xl font-black text-white mb-1 group-hover:text-yellow-400 transition">{{ kelasData.kelas }}</h3>
              <p class="text-blue-300/80 text-sm font-semibold mb-6">{{ kelasData.bodi }} | {{ kelasData.sasis }}</p>
              
              <p class="text-xs font-bold text-slate-500 uppercase tracking-widest mb-4">Fasilitas Mewah:</p>
              <ul class="space-y-3">
                <li v-for="fitur in kelasData.fitur" :key="fitur" class="flex items-center text-slate-300 font-medium">
                  <span class="text-yellow-400 mr-3">✦</span> {{ fitur }}
                </li>
              </ul>
              
              <button @click="gantiHalaman('home')" class="w-full mt-10 bg-slate-900 border border-blue-800 hover:bg-yellow-400 hover:text-blue-950 hover:border-yellow-400 text-white font-black py-4 px-4 rounded-xl transition duration-300">
                CARI JADWAL
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 2. HALAMAN HASIL PENCARIAN ================= -->
      <div v-else-if="halamanSekarang === 'hasil-pencarian'" key="hasil" class="max-w-5xl mx-auto px-4 py-12">
        <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-10 border-b border-blue-900/40 pb-6">
          <div>
            <h2 class="text-3xl font-extrabold text-white mb-2">Jadwal Keberangkatan</h2>
            <p class="text-slate-400 font-medium flex items-center gap-2">
              <span class="bg-blue-950/60 border border-blue-900/50 px-3 py-1 rounded-lg text-white">{{ asal }}</span> 
              <span class="text-yellow-400">➔</span> 
              <span class="bg-blue-950/60 border border-blue-900/50 px-3 py-1 rounded-lg text-white">{{ tujuan }}</span>
              <span class="mx-2 text-slate-600">|</span>
              <span>{{ tanggal }}</span>
            </p>
          </div>
          <button @click="gantiHalaman('home')" class="mt-4 md:mt-0 text-slate-400 hover:text-yellow-400 font-bold transition flex items-center gap-1">
            <span>«</span> Ubah Pencarian
          </button>
        </div>

        <div v-if="jadwalTersaring.length > 0" class="space-y-6">
          <div v-for="bus in jadwalTersaring" :key="bus.id" class="bg-blue-950/30 rounded-3xl border border-blue-900/50 overflow-hidden hover:border-yellow-400/50 transition duration-300 group">
            <div class="flex flex-col md:flex-row">
              <div class="p-6 md:w-1/3 border-b md:border-b-0 md:border-r border-blue-900/50 bg-slate-950/40 flex flex-col justify-center relative overflow-hidden">
                <div class="absolute top-0 right-0 w-24 h-24 bg-yellow-400/10 rounded-full blur-2xl -mr-10 -mt-10"></div>
                <div class="flex justify-between items-center mb-3">
                  <span class="text-4xl font-black text-white">{{ bus.waktuBerangkat }}</span>
                  <span class="text-blue-900/80 font-bold text-xl">➔</span>
                  <span class="text-2xl font-bold text-slate-400">{{ bus.waktuTiba }}</span>
                </div>
                <div class="inline-block bg-yellow-400/10 border border-yellow-400/30 text-yellow-400 text-xs tracking-widest uppercase font-black px-4 py-1.5 rounded-full w-max mt-2">
                  {{ bus.kelas }}
                </div>
              </div>

              <div class="p-6 md:w-1/3 flex flex-col justify-center">
                <h3 class="font-black text-xl text-white mb-2">{{ bus.nomorLambung }}</h3>
                <div class="space-y-1 mb-4">
                  <p class="text-slate-400 text-sm">Bodi: <span class="text-slate-200 font-semibold">{{ bus.bodi }}</span></p>
                  <p class="text-slate-400 text-sm">Sasis: <span class="text-slate-200 font-semibold">{{ bus.sasis }}</span></p>
                </div>
                <button @click="lihatDetail(bus)" class="text-left text-sm text-yellow-400 hover:text-yellow-300 font-bold w-max transition">
                  Lihat Fasilitas Lengkap
                </button>
              </div>

              <div class="p-6 md:w-1/3 flex flex-col justify-center items-end bg-slate-950/20">
                <div class="text-right mb-5">
                  <p class="text-slate-500 text-sm font-bold uppercase tracking-wider mb-1">Total Tiket</p>
                  <p class="text-3xl font-black text-white">{{ formatRupiah(bus.harga) }}</p>
                  <p class="text-sm font-medium mt-2 flex items-center justify-end gap-2">
                    <span class="w-2 h-2 rounded-full" :class="bus.sisaSeat < 5 ? 'bg-red-500' : 'bg-emerald-500'"></span>
                    <span :class="bus.sisaSeat < 5 ? 'text-red-400' : 'text-slate-400'">
                      Sisa {{ bus.sisaSeat }} Kursi ({{ bus.konfigurasi }})
                    </span>
                  </p>
                </div>
                <button @click="lanjutPilihKursi(bus)" class="w-full bg-blue-950/50 border border-blue-800 hover:bg-yellow-400 hover:text-blue-950 text-white font-black py-3.5 px-4 rounded-xl transition duration-300 transform group-hover:scale-[1.02]">
                  PILIH KURSI
                </button>
              </div>
            </div>
          </div>
        </div>

        <div v-else class="bg-blue-950/30 rounded-3xl border border-blue-900/50 p-12 text-center mt-6 shadow-xl">
          <div class="text-7xl mb-6 opacity-30 grayscale">🚌💨</div>
          <h3 class="text-3xl font-black text-white mb-3">Jadwal Tidak Ditemukan</h3>
          <p class="text-slate-400 font-medium mb-8 max-w-md mx-auto">
            Armada Kecapi Trans untuk rute <span class="text-yellow-400 font-bold">{{ asal }} ➔ {{ tujuan }}</span> belum tersedia di tanggal tersebut.
          </p>
          <button @click="gantiHalaman('home')" class="bg-blue-950/60 hover:bg-blue-900 border border-blue-800 text-white font-bold py-3 px-8 rounded-xl transition duration-300">
            Cari Rute Lain
          </button>
        </div>
      </div>

      <!-- ================= 3. HALAMAN PILIH KURSI ================= -->
      <div v-else-if="halamanSekarang === 'pilih-kursi'" key="kursi" class="max-w-5xl mx-auto px-4 py-12">
        <button @click="gantiHalaman('hasil-pencarian')" class="mb-8 text-slate-400 hover:text-yellow-400 font-bold transition flex items-center gap-2">
          <span>«</span> Kembali ke Daftar Jadwal
        </button>

        <div class="flex flex-col md:flex-row gap-8">
          <div class="md:w-2/3 flex justify-center">
            <div class="bg-blue-950/40 border border-blue-900/50 rounded-[3rem] p-8 pb-16 w-max relative shadow-2xl overflow-hidden">
              <div class="absolute top-0 left-1/2 -translate-x-1/2 w-32 h-32 bg-yellow-400/10 blur-[50px] rounded-full pointer-events-none"></div>

              <div class="absolute top-6 right-8 w-14 h-14 border-4 border-blue-900/60 rounded-full flex items-center justify-center opacity-50">
                <div class="w-8 h-1 bg-blue-900/60 transform rotate-45"></div>
                <div class="w-8 h-1 bg-blue-900/60 transform -rotate-45 absolute"></div>
              </div>
              <div class="absolute top-8 left-0 w-2 h-20 bg-blue-900/40 border-r border-blue-900/60 rounded-r-lg"></div>

              <div class="mt-24 relative z-10">
                <div class="grid gap-4" :class="{
                  'grid-cols-5': busPilihan.konfigurasi === '2-2',
                  'grid-cols-4': busPilihan.konfigurasi === '2-1',
                  'grid-cols-3': busPilihan.konfigurasi === '1-1'
                }">
                  <template v-for="(item, index) in susunanKursi" :key="index">
                    <div v-if="item.tipe === 'lorong'" class="w-6 md:w-10"></div>
                    <div v-else-if="item.tipe === 'kosong'" class="w-14 h-14 md:w-16 md:h-16"></div>
                    <div v-else @click="pilihKursiIni(item)" 
                         class="w-14 h-14 md:w-16 md:h-16 rounded-2xl flex flex-col items-center justify-center font-black cursor-pointer transition-all duration-200 transform hover:scale-105 shadow-lg"
                         :class="kursiYangDipilih.includes(item.id) 
                            ? 'bg-yellow-400 text-blue-950 shadow-[0_0_15px_rgba(250,204,21,0.5)] border border-yellow-200' 
                            : 'bg-slate-900 border border-blue-800 hover:bg-slate-800 text-blue-200'">
                      <span class="text-[10px] font-bold tracking-wider uppercase opacity-60 mb-0.5">Seat</span>
                      <span class="text-xl">{{ item.id }}</span>
                    </div>
                  </template>
                </div>
              </div>
            </div>
          </div>

          <div class="md:w-1/3">
            <div class="bg-blue-950/40 rounded-3xl border border-blue-900/50 p-8 sticky top-24 shadow-xl">
              <h3 class="text-2xl font-black text-white mb-1">{{ busPilihan.nomorLambung }}</h3>
              <p class="text-slate-400 font-medium mb-8 border-b border-blue-900/50 pb-5">
                Kelas <span class="text-yellow-400">{{ busPilihan.kelas }}</span> ({{ busPilihan.konfigurasi }})
              </p>

              <div class="mb-6">
                <p class="text-sm font-bold text-blue-200/70 uppercase tracking-widest mb-3">Kursi Terpilih ({{ jumlahKursi }})</p>
                <div v-if="kursiYangDipilih.length === 0" class="text-slate-500 font-medium text-sm italic">
                  Belum ada kursi yang dipilih
                </div>
                <div v-else class="flex flex-wrap gap-2">
                  <span v-for="k in kursiYangDipilih" :key="k" class="bg-yellow-400/10 text-yellow-400 border border-yellow-400/30 font-black px-4 py-2 rounded-xl">
                    {{ k }}
                  </span>
                </div>
              </div>

              <div class="mt-8 border-t border-blue-900/50 pt-6">
                <p class="text-sm font-bold text-blue-200/70 uppercase tracking-widest mb-2">Total Tiket</p>
                <p class="text-4xl font-black text-white mb-8">
                  {{ formatRupiah(subtotal) }}
                </p>
              </div>

              <button @click="bukaPembayaran" 
                      class="w-full font-black text-lg py-4 px-4 rounded-2xl transition-all duration-300"
                      :disabled="kursiYangDipilih.length < parseInt(jumlahKursi)"
                      :class="kursiYangDipilih.length === parseInt(jumlahKursi) 
                        ? 'bg-gradient-to-r from-yellow-500 to-yellow-400 text-blue-950 shadow-[0_0_20px_rgba(250,204,21,0.3)] hover:scale-[1.02]' 
                        : 'bg-blue-950/50 text-slate-500 border border-blue-900/50 cursor-not-allowed'">
                LANJUT BAYAR
              </button>
              <p v-if="kursiYangDipilih.length < parseInt(jumlahKursi)" class="text-xs text-center text-slate-400 mt-4 font-medium">
                Silakan pilih <span class="text-yellow-400">{{ parseInt(jumlahKursi) - kursiYangDipilih.length }} kursi</span> lagi di denah.
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 4. HALAMAN PILIH METODE PEMBAYARAN ================= -->
      <div v-else-if="halamanSekarang === 'pilih-metode-pembayaran'" key="pembayaran" class="max-w-6xl mx-auto px-4 py-12">
        <button @click="gantiHalaman('pilih-kursi')" class="mb-8 text-slate-400 hover:text-yellow-400 font-bold transition flex items-center gap-2">
          <span>«</span> Kembali Pilih Kursi
        </button>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div class="md:col-span-2 space-y-8">
            <div class="bg-blue-950/40 rounded-3xl border border-blue-900/50 p-8 shadow-2xl relative overflow-hidden">
              <div class="h-1 w-full absolute top-0 left-0 bg-yellow-400"></div>
              <h2 class="text-3xl font-black text-white mb-8">Data Penumpang & Lokasi</h2>
              
              <form @submit.prevent="lanjutBayarDetail" class="space-y-6">
                
                <div v-for="(penumpang, index) in dataPenumpang" :key="index" class="p-6 border border-blue-900/50 rounded-2xl bg-slate-950/40 mb-6 hover:border-yellow-900 transition">
                  <h4 class="text-yellow-400 font-extrabold mb-4 uppercase tracking-wider text-sm flex items-center gap-2">
                    <span class="bg-yellow-400/10 text-yellow-400 border border-yellow-400/30 px-3 py-1 rounded-lg">Penumpang {{ index + 1 }}</span>
                    <span class="text-slate-400">| Seat: {{ kursiYangDipilih[index] }}</span>
                  </h4>
                  
                  <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div>
                      <label class="block text-xs font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Nama Lengkap</label>
                      <input type="text" v-model="penumpang.nama" placeholder="Sesuai KTP" class="w-full border border-blue-900/60 rounded-xl p-3 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white bg-slate-950/50 placeholder-slate-500" />
                    </div>
                    <div>
                      <label class="block text-xs font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Servis Makan Premium</label>
                      <select v-model="penumpang.makan" class="w-full border border-blue-900/60 rounded-xl p-3 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white bg-slate-950/50 appearance-none">
                        <option value="" disabled selected>Pilih Menu Makan</option>
                        <option value="Nasi Rendang Premium">Nasi Rendang Premium</option>
                        <option value="Ayam Bakar Taliwang">Ayam Bakar Taliwang</option>
                        <option value="Nasi Goreng Spesial">Nasi Goreng Spesial</option>
                      </select>
                    </div>
                  </div>
                </div>

                <hr class="border-blue-900/50">

                <div class="grid grid-cols-1 md:grid-cols-2 gap-6 pt-2">
                  <div class="md:col-span-2">
                    <label class="block text-sm font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Nomor WhatsApp Pemesan</label>
                    <input type="number" v-model="noHp" placeholder="0812xxxxxx (Untuk kirim E-Tiket)" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white font-medium bg-slate-950/50 placeholder-slate-500" />
                  </div>
                  <div>
                    <label class="block text-sm font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Titik Naik ({{ busPilihan.asal }})</label>
                    <select v-model="titikNaik" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 outline-none text-white font-medium bg-slate-950/50 appearance-none">
                      <option value="" disabled selected>Pilih Lokasi Naik</option>
                      <option v-for="lokasi in daftarTitik[busPilihan.asal]" :key="lokasi" :value="lokasi">{{ lokasi }}</option>
                    </select>
                  </div>
                  <div>
                    <label class="block text-sm font-bold text-blue-200/70 mb-2 uppercase tracking-wide">Titik Turun ({{ busPilihan.tujuan }})</label>
                    <select v-model="titikTurun" class="w-full border border-blue-900/60 rounded-2xl p-4 focus:ring-2 focus:ring-yellow-400 outline-none text-white font-medium bg-slate-950/50 appearance-none">
                      <option value="" disabled selected>Pilih Lokasi Turun</option>
                      <option v-for="lokasi in daftarTitik[busPilihan.tujuan]" :key="lokasi" :value="lokasi">{{ lokasi }}</option>
                    </select>
                  </div>
                </div>
              </form>
            </div>

            <div class="bg-blue-950/40 rounded-3xl border border-blue-900/50 p-8 shadow-2xl">
              <h3 class="text-xl font-bold text-blue-200/70 mb-6 uppercase tracking-wider">Metode Pembayaran</h3>
              <div class="space-y-4">
                <label class="flex items-center gap-4 bg-slate-950/40 p-5 rounded-xl border border-blue-900/50 cursor-pointer hover:border-yellow-400/50 transition" :class="metodePembayaran === 'QRIS' ? 'border-yellow-400 bg-yellow-400/10' : ''">
                  <input type="radio" v-model="metodePembayaran" value="QRIS" class="w-5 h-5 text-yellow-400 [color-scheme:dark]">
                  <span class="text-white font-bold text-lg">QRIS (Gopay, OVO, Dana, dll)</span>
                </label>
                <label class="flex items-center gap-4 bg-slate-950/40 p-5 rounded-xl border border-blue-900/50 cursor-pointer hover:border-yellow-400/50 transition" :class="metodePembayaran === 'Transfer Bank' ? 'border-yellow-400 bg-yellow-400/10' : ''">
                  <input type="radio" v-model="metodePembayaran" value="Transfer Bank" class="w-5 h-5 text-yellow-400 [color-scheme:dark]">
                  <span class="text-white font-bold text-lg">Transfer Virtual Account (BCA, Mandiri, BRI)</span>
                </label>
              </div>
            </div>
          </div>

          <div class="md:col-span-1">
            <div class="space-y-8 sticky top-24">
                <!-- RINCIAN HARGA -->
                <div class="bg-blue-950/40 rounded-3xl border border-blue-900/60 p-8 shadow-[0_0_30px_rgba(250,204,21,0.1)] relative overflow-hidden">
                <div class="absolute top-0 right-0 w-32 h-32 bg-yellow-400/10 blur-[60px] rounded-full pointer-events-none"></div>

                <h3 class="text-xl font-black text-white mb-1">{{ busPilihan.nomorLambung }}</h3>
                <p class="text-slate-400 font-medium mb-6">{{ busPilihan.asal }} ➔ {{ busPilihan.tujuan }}</p>

                <div class="space-y-4 border-t border-blue-900/50 pt-6">
                    <div class="flex justify-between text-slate-400">
                    <span>Tiket (x{{ jumlahKursi }})</span>
                    <span class="font-semibold text-white">{{ formatRupiah(subtotal) }}</span>
                    </div>
                    <div class="flex justify-between text-slate-400">
                    <span>Biaya Admin</span>
                    <span class="font-semibold text-emerald-400">{{ formatRupiah(adminFee) }}</span>
                    </div>
                    
                    <div v-if="discount > 0" class="flex justify-between text-slate-400 pb-4 border-b border-blue-900/50">
                    <span>Diskon Spesial 🎟️</span>
                    <span class="font-semibold text-yellow-400">- {{ formatRupiah(discount) }}</span>
                    </div>
                    <div v-else class="pb-4 border-b border-blue-900/50"></div>

                    <div class="flex justify-between pt-4">
                    <span class="text-xl font-black text-white uppercase tracking-wider">Total Akhir</span>
                    <span class="text-3xl font-black text-yellow-400">{{ formatRupiah(totalAkhir) }}</span>
                    </div>
                </div>

                <button @click="lanjutBayarDetail" class="w-full mt-10 bg-gradient-to-r from-yellow-500 to-yellow-400 hover:from-yellow-400 hover:to-yellow-300 text-blue-950 font-black text-lg p-4 rounded-xl shadow-[0_0_20px_rgba(250,204,21,0.3)] transition duration-300 transform hover:-translate-y-1">
                    BAYAR SEKARANG
                </button>
                </div>

                <!-- KOTAK KODE PROMO -->
                <div class="bg-slate-900 rounded-3xl border border-blue-800 p-7 shadow-xl">
                    <label class="block text-sm font-bold text-blue-200/70 mb-3 uppercase tracking-wide flex items-center gap-2">
                        <span>🎟️ Punya Kode Promo Yahud?</span>
                    </label>
                    <div class="flex gap-3">
                        <input type="text" v-model="userInputPromoCode" placeholder="Masukin kodenya..." class="flex-grow border border-blue-900 rounded-xl p-3 focus:ring-2 focus:ring-yellow-400 outline-none transition text-white bg-slate-950 placeholder-slate-600 uppercase tracking-wider font-bold" />
                        <button @click="terapkanPromoCode" class="bg-blue-900 hover:bg-blue-800 text-yellow-400 font-bold px-5 rounded-xl transition border border-blue-800 active:scale-95">
                            Pakai
                        </button>
                    </div>
                    <Transition name="fade">
                        <p v-if="promoMessage" class="text-xs mt-3 font-bold px-2" :class="promoStatus === 'applied' ? 'text-emerald-400' : 'text-red-400'">
                            {{ promoMessage }}
                        </p>
                    </Transition>
                </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 5. HALAMAN TIMER MENUNGGU PEMBAYARAN ================= -->
      <div v-else-if="halamanSekarang === 'menunggu-pembayaran'" key="menunggu" class="max-w-xl mx-auto px-4 py-16 text-center">
        <div class="bg-blue-950/40 backdrop-blur-xl rounded-[2.5rem] p-12 shadow-[0_0_60px_rgba(0,0,0,0.4)] border border-blue-900/50 relative">
          <div class="absolute -top-10 left-1/2 -translate-x-1/2 bg-slate-950 p-4 rounded-full border border-blue-900/50 shadow-2xl grayscale">🚌💨</div>
          
          <h2 class="text-3xl font-black text-white mb-6">Menunggu Pembayaran Lo, Sob</h2>
          <p class="text-slate-300 font-medium mb-12">Kursi pilihan lo aman, tapi kudu buruan transfer sesuai pilihan lo sebelum expired!</p>
          
          <div class="bg-slate-950 border border-blue-900/50 p-8 rounded-3xl font-black text-white mb-12 relative overflow-hidden shadow-inner">
            <div class="absolute inset-0 bg-yellow-400/5 blur-[30px] rounded-full pointer-events-none"></div>
            <p class="text-blue-200/70 text-xs font-bold uppercase tracking-widest mb-2">Sisa Waktu</p>
            <div class="text-6xl tracking-widest flex justify-center gap-1 text-yellow-400">
              <span class="bg-blue-950/60 border border-blue-900/50 px-3 rounded-xl shadow-lg">{{ menitSisa.toString().padStart(2, '0') }}</span>
              <span>:</span>
              <span class="bg-blue-950/60 border border-blue-900/50 px-3 rounded-xl shadow-lg">{{ detikSisa.toString().padStart(2, '0') }}</span>
            </div>
            <p class="text-sm text-slate-500 mt-2">Menit</p>
          </div>

          <button @click="konfirmasiBayar" class="w-full mt-4 bg-gradient-to-r from-yellow-500 to-yellow-400 text-blue-950 font-black text-lg p-4 rounded-xl shadow-[0_0_20px_rgba(250,204,21,0.3)] transition duration-300 transform hover:-translate-y-1">
            CEK STATUS PEMBAYARAN
          </button>
        </div>
      </div>

      <!-- ================= 6. HALAMAN PEMBAYARAN BERHASIL ================= -->
      <div v-else-if="halamanSekarang === 'pembayaran-sukses'" key="sukses" class="max-w-xl mx-auto px-4 py-16 text-center">
        <div class="bg-blue-950/40 rounded-[2.5rem] p-12 shadow-[0_0_50px_rgba(0,0,0,0.6)] border border-blue-900/50 text-center">
          <div class="text-9xl mb-10 text-yellow-400 flex justify-center relative">
            <div class="absolute inset-0 bg-yellow-400/20 blur-[60px] rounded-full pointer-events-none"></div>
            <span class="relative">✓</span>
          </div>
          <h2 class="text-4xl font-black text-white mb-6">Pembayaran Berhasil!</h2>
          <p class="text-slate-300 font-medium mb-12">Pesenan tiket Kecapi Trans lo udah dikonfirmasi mesin. Lo dapet tiket premium Yahud!</p>
          
          <p class="text-blue-200/70 text-xs font-bold uppercase tracking-widest mb-2">Booking Code</p>
          <div class="bg-slate-950 border border-blue-900/50 p-6 rounded-2xl font-black text-2xl text-yellow-400 mb-12 tracking-wider shadow-inner">
            {{ bookingCode }}
          </div>

          <button @click="gantiHalaman('lihat-e-tiket')" class="w-full bg-gradient-to-r from-yellow-500 to-yellow-400 hover:from-yellow-400 hover:to-yellow-300 text-blue-950 font-black text-lg p-4 rounded-xl shadow-[0_0_20px_rgba(250,204,21,0.3)] transition duration-300 transform hover:-translate-y-1">
            LIHAT E-TIKET DIGITAL
          </button>
        </div>
      </div>

      <!-- ================= 7. HALAMAN LIHAT E-TIKET ================= -->
      <div v-else-if="halamanSekarang === 'lihat-e-tiket'" key="tiket" class="max-w-2xl mx-auto px-4 py-12">
        <button @click="gantiHalaman('home')" class="mb-8 text-slate-400 hover:text-yellow-400 font-bold transition flex items-center gap-2">
          <span>«</span> Kembali ke Beranda
        </button>

        <div class="bg-blue-950/40 rounded-3xl border border-blue-900/50 overflow-hidden shadow-2xl relative">
          <!-- Aksen Kop Surat Tiket (Navy & Yellow) -->
          <div class="h-2 w-full absolute top-0 left-0 bg-gradient-to-r from-blue-900 via-yellow-400 to-blue-900"></div>
          
          <div class="bg-slate-950 p-6 border-b border-blue-900/50 flex justify-between items-center text-center">
              <h1 class="text-3xl font-black text-white italic tracking-wider">
                KECAPI <span class="text-yellow-400">TRANS</span>
              </h1>
              <span class="text-yellow-400 text-sm font-black bg-yellow-400/10 px-3 py-1 rounded border border-yellow-400/30 uppercase tracking-widest">Valid E-Ticket</span>
          </div>

          <div class="p-8">
            <div class="bg-slate-950 p-6 rounded-2xl border border-blue-900/50 mb-8 flex justify-between items-center relative overflow-hidden">
              <div class="absolute inset-0 bg-yellow-400/5 blur-[50px] pointer-events-none"></div>
              <div>
                <p class="font-bold text-blue-200/70 text-sm mb-1 uppercase tracking-wider">Perjalanan</p>
                <p class="text-2xl font-black text-white">{{ busPilihan?.asal || 'Jakarta' }} <span class="text-yellow-400 mx-2">➔</span> {{ busPilihan?.tujuan || 'Malang' }}</p>
                <p class="text-slate-400 font-medium mt-1">{{ busPilihan?.nomorLambung || 'KCP-101' }} ({{ busPilihan?.kelas || 'Executive' }})</p>
              </div>
              <div class="text-right">
                <p class="text-sm text-blue-200/70 font-bold mb-1 uppercase tracking-wider">Booking Code</p>
                <p class="text-2xl font-black text-yellow-400 tracking-wider">{{ bookingCode }}</p>
              </div>
            </div>

            <!-- DETAIL LOKASI & KONTAK -->
            <div class="grid grid-cols-2 gap-8 border-t border-b border-blue-900/50 py-8 mb-8">
              <div>
                <p class="text-sm font-medium text-blue-200/70 mb-1">Titik Naik</p>
                <p class="text-lg font-extrabold text-white">{{ titikNaik || '-' }}</p>
              </div>
              <div>
                <p class="text-sm font-medium text-blue-200/70 mb-1">Titik Turun</p>
                <p class="text-lg font-extrabold text-white">{{ titikTurun || '-' }}</p>
              </div>
              <div>
                <p class="text-sm font-medium text-blue-200/70 mb-1">WhatsApp Pemesan</p>
                <p class="text-lg font-extrabold text-white">{{ noHp || '-' }}</p>
              </div>
              <div>
                <p class="text-sm font-medium text-blue-200/70 mb-1">Total Kursi</p>
                <p class="text-lg font-extrabold text-white">{{ jumlahKursi }} Kursi</p>
              </div>
            </div>

            <!-- DAFTAR PENUMPANG & SERVIS MAKAN -->
            <div class="mb-8">
              <p class="text-sm font-bold text-blue-200/70 uppercase tracking-widest mb-4">Daftar Penumpang</p>
              <div class="space-y-4">
                <div v-for="(penumpang, index) in dataPenumpang" :key="index" class="bg-slate-950 p-4 rounded-xl border border-blue-900/50 flex justify-between items-center hover:border-yellow-900 transition">
                  <div>
                    <p class="font-extrabold text-white text-lg">{{ penumpang.nama || 'Penumpang ' + (index+1) }}</p>
                    <p class="text-sm text-yellow-400 mt-1">🍽️ {{ penumpang.makan || 'Standar' }}</p>
                  </div>
                  <div class="text-right">
                    <span class="bg-yellow-400/10 text-yellow-400 font-black px-4 py-2 rounded-lg border border-yellow-400/30">Seat {{ kursiYangDipilih[index] || '-' }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- DETAIL KODE PROMO DI TIKET -->
            <div v-if="discount > 0" class="bg-slate-950 p-5 rounded-2xl border-2 border-dashed border-yellow-800 text-center mb-8">
                <p class="text-sm font-bold text-yellow-600 uppercase tracking-widest mb-1">Hemat Pake Promo 🎟️</p>
                <p class="text-xl font-black text-yellow-400">{{ userInputPromoCode.toUpperCase() }}</p>
                <p class="text-xs text-slate-500 mt-1">Potongan harga {{ formatRupiah(discount) }} udah diterapkan.</p>
            </div>

            <p class="text-xs text-center text-slate-500 mt-8 font-medium italic border-t border-blue-900/50 pt-6">
              *Tunjukkan e-tiket ini ke kru Kecapi Trans saat boarding dan klaim servis makan premium lo. Enjoy the trip! 🚌
            </p>
          </div>
        </div>
      </div>

    </Transition>
  </div>
</template>

<style scoped>
/* CSS ANIMASI VUE TRANSTION */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(15px);
}

/* Biar input number gak muncul panah up/down */
input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
  -webkit-appearance: none; 
  margin: 0; 
}
input[type=number] {
  -moz-appearance: textfield;
}
</style>