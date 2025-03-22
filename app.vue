<template>
  <div class="min-h-screen bg-gray-100 p-4">
    <div class="max-w-7xl mx-auto">
      <h1 class="text-3xl font-bold text-center my-6 text-gray-800">WhatsApp Bulk Sender</h1>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <!-- Kolom Kiri: Input dan Preview -->
        <div class="space-y-6">
          <!-- Form Input -->
          <div class="bg-white p-6 rounded-lg shadow-md">
            <h2 class="text-xl font-semibold mb-4">Input Daftar Tamu</h2>

            <form
              id="guestForm"
              @submit.prevent="addGuest"
              class="mb-6">
              <div class="grid grid-cols-1 gap-4 mb-4">
                <div>
                  <label
                    for="guestName"
                    class="block text-sm font-medium text-gray-700 mb-1">
                    Nama Tamu
                  </label>
                  <input
                    id="guestName"
                    v-model="newGuest.guest_name"
                    type="text"
                    required
                    placeholder="Masukkan nama tamu"
                    class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500" />
                </div>

                <div>
                  <label
                    for="phoneNumber"
                    class="block text-sm font-medium text-gray-700 mb-1">
                    Nomor WhatsApp
                  </label>
                  <input
                    id="phoneNumber"
                    v-model="newGuest.phone_number"
                    type="text"
                    required
                    placeholder="Contoh: 08123456789 atau 628123456789"
                    class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500" />
                  <p class="text-xs text-gray-500 mt-1">Nomor yang diawali 0 akan otomatis dikonversi ke format 62</p>
                </div>
              </div>

              <div class="flex gap-2">
                <button
                  type="submit"
                  class="flex-1 px-4 py-2 text-white rounded focus:outline-none focus:ring-2 transition duration-200"
                  :class="editMode ? 'bg-green-600 hover:bg-green-700 focus:ring-green-500' : 'bg-blue-600 hover:bg-blue-700 focus:ring-blue-500'">
                  {{ editMode ? 'Simpan Perubahan' : 'Tambah Tamu' }}
                </button>

                <button
                  v-if="editMode"
                  type="button"
                  @click="cancelEdit"
                  class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 transition duration-200">
                  Batal
                </button>
              </div>
            </form>

            <div class="border-t border-gray-200 pt-4">
              <div class="flex justify-between items-center mb-4">
                <h3 class="font-medium">Atau Import Data</h3>
                <button
                  @click="loadSampleData"
                  class="text-sm text-blue-600 hover:text-blue-800">
                  Load Sample Data
                </button>
              </div>

              <textarea
                v-model="guestsInput"
                placeholder="Paste JSON data tamu di sini..."
                class="w-full h-36 p-3 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500 font-mono text-sm"></textarea>
              <div class="mt-4">
                <button
                  @click="parseGuests"
                  class="w-full px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 transition duration-200">
                  Import Data
                </button>
              </div>

              <div
                v-if="error"
                class="mt-3 text-red-600">
                {{ error }}
              </div>
            </div>
          </div>

          <!-- Template Pesan -->
          <div class="bg-white p-6 rounded-lg shadow-md">
            <h2 class="text-xl font-semibold mb-4">Template Pesan</h2>
            <textarea
              v-model="messageTemplate"
              class="w-full h-60 p-3 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"></textarea>
          </div>

          <!-- Preview & Sender -->
          <div class="bg-white p-6 rounded-lg shadow-md">
            <h2 class="text-xl font-semibold mb-4">Preview Template</h2>
            <div
              v-if="guests.length > 0"
              class="mb-4">
              <select
                v-model="selectedGuest"
                class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500">
                <option
                  v-for="(guest, index) in guests"
                  :key="index"
                  :value="index">
                  {{ guest.guest_name }} ({{ guest.phone_number }}) - {{ guest.is_send ? 'Terkirim' : 'Belum Terkirim' }}
                </option>
              </select>
            </div>

            <div
              v-if="selectedGuest !== null"
              class="bg-green-100 p-4 rounded border border-green-300 whitespace-pre-line">
              {{ getFormattedMessage(guests[selectedGuest]) }}
            </div>

            <div
              v-else
              class="bg-gray-100 p-4 rounded border border-gray-300 text-gray-500 text-center">
              Belum ada tamu yang dipilih untuk preview
            </div>
          </div>
        </div>

        <!-- Kolom Kanan: Daftar Tamu -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-semibold">Daftar Tamu ({{ guests.length }})</h2>
            <div class="flex items-center gap-2">
              <span class="mr-2 hidden md:inline">Terkirim: {{ sentCount }}/{{ guests.length }}</span>
              <div class="flex flex-wrap gap-2">
                <button
                  @click="sendAll"
                  class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 transition duration-200"
                  :disabled="guests.length === 0">
                  Kirim Semua
                </button>
                <button
                  @click="clearAllData"
                  class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 transition duration-200"
                  :disabled="guests.length === 0">
                  Hapus Semua
                </button>
              </div>
            </div>
          </div>

          <div class="overflow-hidden border border-gray-200 rounded-lg">
            <div class="overflow-x-auto">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-50">
                  <tr>
                    <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                    <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama</th>
                    <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nomor WhatsApp</th>
                    <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                    <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Aksi</th>
                  </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">
                  <tr v-if="guests.length === 0">
                    <td
                      colspan="5"
                      class="px-3 py-4 text-center text-sm text-gray-500">
                      Belum ada data tamu. Tambahkan tamu baru atau import data.
                    </td>
                  </tr>
                  <tr
                    v-for="(guest, index) in guests"
                    :key="index"
                    :class="guest.is_send ? 'bg-green-50' : ''">
                    <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-900">{{ index + 1 }}</td>
                    <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-900">{{ guest.guest_name }}</td>
                    <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-900">{{ guest.phone_number }}</td>
                    <td class="px-3 py-4 whitespace-nowrap text-sm">
                      <span :class="guest.is_send ? 'bg-green-100 text-green-800 px-2 py-1 rounded text-xs font-medium' : 'bg-yellow-100 text-yellow-800 px-2 py-1 rounded text-xs font-medium'">
                        {{ guest.is_send ? 'Terkirim' : 'Belum terkirim' }}
                      </span>
                    </td>
                    <td class="px-3 py-4 whitespace-nowrap text-sm">
                      <div class="flex gap-2">
                        <button
                          @click="sendMessage(guest, index)"
                          class="text-blue-600 hover:text-blue-900 rounded-full p-1 hover:bg-blue-100 transition duration-200"
                          :disabled="guest.is_send"
                          :class="guest.is_send ? 'opacity-50 cursor-not-allowed' : ''"
                          title="Kirim pesan">
                          <span class="sr-only">{{ guest.is_send ? 'Terkirim' : 'Kirim' }}</span>
                          <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="h-5 w-5"
                            viewBox="0 0 20 20"
                            fill="currentColor">
                            <path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z" />
                          </svg>
                        </button>
                        <button
                          @click="editGuest(index)"
                          class="text-yellow-600 hover:text-yellow-900 rounded-full p-1 hover:bg-yellow-100 transition duration-200"
                          title="Edit data tamu">
                          <span class="sr-only">Edit</span>
                          <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="h-5 w-5"
                            viewBox="0 0 20 20"
                            fill="currentColor">
                            <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
                          </svg>
                        </button>
                        <button
                          @click="removeGuest(index)"
                          class="text-red-600 hover:text-red-900 rounded-full p-1 hover:bg-red-100 transition duration-200"
                          title="Hapus tamu">
                          <span class="sr-only">Hapus</span>
                          <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="h-5 w-5"
                            viewBox="0 0 20 20"
                            fill="currentColor">
                            <path
                              fill-rule="evenodd"
                              d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                              clip-rule="evenodd" />
                          </svg>
                        </button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <!-- Mobile stats counter -->
          <div class="mt-4 text-center md:hidden">
            <span class="font-medium">Terkirim: {{ sentCount }}/{{ guests.length }}</span>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <div class="mt-8 text-center text-gray-500 text-sm">&copy; 2025 WhatsApp Bulk Sender | Dibuat dengan Nuxt 3 & Tailwind CSS</div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed, onMounted, watch } from 'vue';

  // Define interface untuk tamu
  interface Guest {
    guest_name: string;
    phone_number: string;
    is_send: boolean;
  }

  // Form data untuk tamu baru
  const newGuest = ref<Guest>({
    guest_name: '',
    phone_number: '',
    is_send: false,
  });

  // Mode edit
  const editMode = ref<boolean>(false);
  const editIndex = ref<number>(-1);

  // Data guests
  const guestsInput = ref<string>('');
  const guests = ref<Guest[]>([]);
  const error = ref<string>('');
  const selectedGuest = ref<number | null>(null);

  // Template pesan
  const messageTemplate = ref<string>(`Kepada Yth.
Bapak/Ibu/Saudara/i
{{guest_name}}
di tempat

Tanpa mengurangi rasa hormat, perkenankan kami mengundang Bapak/Ibu/Saudara/i, untuk menghadiri acara Resepsi Pernikahan Kami

Info lebih lengkap klik link dibawah ini
https://satumomen.com/eka-dan-indra?to={{encoded_guest_name}}

Merupakan suatu kebahagiaan bagi kami apabila Bapak/Ibu/Saudara/i berkenan untuk hadir dan memberikan doa restu.

Kami yang berbahagia
Eka & Indra

Mohon maaf perihal undangan hanya dibagikan melalui pesan ini.`);

  // Jumlah yang sudah terkirim
  const sentCount = computed((): number => {
    return guests.value.filter(guest => guest.is_send).length;
  });

  // Function untuk load sample data
  const loadSampleData = (): void => {
    guestsInput.value = JSON.stringify(
      [
        {
          guest_name: 'Daiv William',
          phone_number: '082192089334',
          is_send: false,
        },
        {
          guest_name: 'Wilhel Setligh',
          phone_number: '082292192488',
          is_send: true,
        },
        {
          guest_name: 'Andre Taulany',
          phone_number: '082192553488',
          is_send: false,
        },
      ],
      null,
      2
    );
  };

  // Parse data tamu
  const parseGuests = (): void => {
    try {
      const parsed = JSON.parse(guestsInput.value);
      if (!Array.isArray(parsed)) {
        error.value = 'Format data harus berupa array';
        return;
      }

      // Validasi format data
      const isValid = parsed.every((item: any) => typeof item === 'object' && 'guest_name' in item && 'phone_number' in item);

      if (!isValid) {
        error.value = 'Format data tidak valid, pastikan setiap item memiliki guest_name dan phone_number';
        return;
      }

      // Set data yang sudah valid dengan format nomor telepon
      guests.value = parsed.map((guest: any): Guest => {
        // Format nomor telepon (konversi 0 ke 62 di awal)
        let formattedPhone = guest.phone_number.trim();
        if (formattedPhone.startsWith('0')) {
          formattedPhone = '62' + formattedPhone.substring(1);
        }

        return {
          guest_name: guest.guest_name,
          phone_number: formattedPhone,
          is_send: guest.is_send || false,
        };
      });

      // Reset error jika berhasil
      error.value = '';

      // Set selected guest ke pertama jika ada
      if (guests.value.length > 0) {
        selectedGuest.value = 0;
      }

      // Save to cache after import
      saveDataToCache();
    } catch (e: any) {
      error.value = `Error parsing JSON: ${e.message}`;
    }
  };

  // Function untuk mendapatkan pesan yang sudah diformatting
  const getFormattedMessage = (guest: Guest): string => {
    if (!guest) return '';

    return messageTemplate.value.replace(/{{guest_name}}/g, guest.guest_name).replace(/{{encoded_guest_name}}/g, encodeURIComponent(guest.guest_name));
  };

  // Function untuk mengirim pesan WhatsApp
  const sendMessage = (guest: Guest, index: number): void => {
    if (guest.is_send) return;

    const message = getFormattedMessage(guest);
    const encodedMessage = encodeURIComponent(message);

    // Pastikan format nomor telepon benar (62xxx)
    let phoneNumber = guest.phone_number.replace(/\D/g, '');
    if (phoneNumber.startsWith('0')) {
      phoneNumber = '62' + phoneNumber.substring(1);
    } else if (!phoneNumber.startsWith('62') && !phoneNumber.startsWith('+62')) {
      // Jika tidak dimulai dengan 62 atau +62, tambahkan 62 di depan
      phoneNumber = '62' + phoneNumber;
    }

    const whatsappUrl = `https://wa.me/${phoneNumber}?text=${encodedMessage}`;

    // Buka link WhatsApp
    window.open(whatsappUrl, '_blank');

    // Update status terkirim
    guests.value[index].is_send = true;

    // Update nomor telepon jika berubah format
    guests.value[index].phone_number = phoneNumber;

    // Save to cache after sending
    saveDataToCache();
  };

  // Fungsi untuk menghapus tamu dari daftar
  const removeGuest = (index: number): void => {
    if (confirm('Apakah Anda yakin ingin menghapus tamu ini?')) {
      guests.value.splice(index, 1);

      // Update selectedGuest jika perlu
      if (guests.value.length === 0) {
        selectedGuest.value = null;
      } else if (selectedGuest.value !== null && selectedGuest.value >= index) {
        // Jika tamu yang dihapus adalah yang dipilih atau sebelumnya, update selected index
        selectedGuest.value = Math.max(0, selectedGuest.value - 1);
      }

      // Save changes to cache
      saveDataToCache();
    }
  };

  // Mengirim ke semua tamu yang belum terkirim
  const sendAll = (): void => {
    guests.value.forEach((guest, index) => {
      if (!guest.is_send) {
        setTimeout(() => {
          sendMessage(guest, index);
        }, index * 1000); // Delay 1 detik antara pengiriman
      }
    });
  };

  // Fungsi untuk menghapus semua data
  const clearAllData = (): void => {
    if (confirm('Apakah Anda yakin ingin menghapus semua data tamu?')) {
      guests.value = [];
      selectedGuest.value = null;
      localStorage.removeItem('whatsapp-guests');
    }
  };

  // Fungsi untuk menambah tamu baru dari form
  const addGuest = (): void => {
    // Validasi nomor telepon
    const phoneRegex = /^[0-9+\-\s]+$/;
    if (!phoneRegex.test(newGuest.value.phone_number)) {
      error.value = 'Nomor telepon hanya boleh berisi angka, +, - atau spasi';
      return;
    }

    // Format nomor telepon (konversi 0 ke 62 di awal)
    let formattedPhone = newGuest.value.phone_number.trim();
    if (formattedPhone.startsWith('0')) {
      formattedPhone = '62' + formattedPhone.substring(1);
    }

    if (editMode.value && editIndex.value >= 0) {
      // Update data tamu yang sedang diedit
      guests.value[editIndex.value] = {
        guest_name: newGuest.value.guest_name,
        phone_number: formattedPhone,
        is_send: guests.value[editIndex.value].is_send, // Pertahankan status pengiriman
      };

      // Reset mode edit
      editMode.value = false;
      editIndex.value = -1;
    } else {
      // Tambahkan tamu baru ke daftar
      guests.value.push({
        guest_name: newGuest.value.guest_name,
        phone_number: formattedPhone,
        is_send: false,
      });
    }

    // Reset form
    newGuest.value = {
      guest_name: '',
      phone_number: '',
      is_send: false,
    };

    // Reset error
    error.value = '';

    // Update selectedGuest jika belum diset
    if (selectedGuest.value === null && guests.value.length > 0) {
      selectedGuest.value = 0;
    }

    // Simpan ke cache
    saveDataToCache();
  };

  // Fungsi untuk membatalkan proses edit
  const cancelEdit = (): void => {
    editMode.value = false;
    editIndex.value = -1;

    // Reset form
    newGuest.value = {
      guest_name: '',
      phone_number: '',
      is_send: false,
    };

    // Reset error
    error.value = '';
  };

  // Fungsi untuk mulai mode edit
  const editGuest = (index: number): void => {
    const guest = guests.value[index];

    // Set form dengan data tamu yang akan diedit
    newGuest.value = {
      guest_name: guest.guest_name,
      phone_number: guest.phone_number,
      is_send: guest.is_send,
    };

    // Set mode edit
    editMode.value = true;
    editIndex.value = index;

    // Scroll ke form
    document.getElementById('guestForm')?.scrollIntoView({ behavior: 'smooth' });
  };

  // Fungsi untuk menyimpan data ke localStorage
  const saveDataToCache = (): void => {
    localStorage.setItem('whatsapp-guests', JSON.stringify(guests.value));
  };

  // Fungsi untuk mengambil data dari localStorage
  const loadDataFromCache = (): void => {
    try {
      const cachedData = localStorage.getItem('whatsapp-guests');
      if (cachedData) {
        guests.value = JSON.parse(cachedData);

        // Set selected guest ke pertama jika ada
        if (guests.value.length > 0 && selectedGuest.value === null) {
          selectedGuest.value = 0;
        }

        console.log('Data loaded from cache:', guests.value.length, 'guests');
      }
    } catch (e) {
      console.error('Error loading data from cache:', e);
    }
  };

  // Watch guests untuk menyimpan perubahan ke cache
  watch(
    guests,
    () => {
      saveDataToCache();
    },
    { deep: true }
  );

  // Load data dari cache saat komponen dimount
  onMounted((): void => {
    loadDataFromCache();
  });
</script>
