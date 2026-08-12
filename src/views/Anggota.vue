<template>
  <div class="space-y-8">
    <!-- ===== AKUN INTERN (khusus Super Admin) ===== -->
    <div v-if="isSuperAdmin">
      <h1 class="text-xl sm:text-2xl font-bold mb-4 dark:text-slate-100">Akun Intern</h1>

      <div v-if="loadingIntern" class="text-sm text-gray-400 dark:text-slate-500 py-4">Memuat...</div>

      <div v-else>
        <!-- Tampilan tabel (desktop/tablet ke atas) -->
        <div class="hidden sm:block bg-white dark:bg-slate-800 rounded-xl border border-gray-200 dark:border-slate-700 overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="bg-gray-50 dark:bg-slate-700/50">
              <tr class="text-left text-gray-500 dark:text-slate-400">
                <th class="p-3">Nama</th><th>Jabatan</th><th>Divisi</th><th>Status</th><th>QR</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="a in anggotaIntern" :key="a.id" class="border-t border-gray-100 dark:border-slate-700 dark:text-slate-200">
                <td class="p-3">{{ a.nama }}</td>
                <td>{{ a.jabatan || '-' }}</td>
                <td>{{ a.divisi || '-' }}</td>
                <td>
                  <span class="px-2 py-1 rounded-full text-xs"
                    :class="a.status_keanggotaan === 'aktif'
                      ? 'bg-green-100 text-green-700 dark:bg-emerald-900/40 dark:text-emerald-300'
                      : 'bg-gray-200 text-gray-600 dark:bg-slate-700 dark:text-slate-400'">
                    {{ a.status_keanggotaan }}
                  </span>
                </td>
                <td>
                  <qrcode-vue :value="a.nomor_anggota || String(a.id)" :size="48" />
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Tampilan kartu (mobile) -->
        <div class="sm:hidden space-y-3">
          <div v-for="a in anggotaIntern" :key="a.id"
            class="bg-white dark:bg-slate-800 border border-gray-200 dark:border-slate-700 rounded-xl p-4">
            <div class="flex justify-between items-start gap-3">
              <div class="min-w-0">
                <p class="font-medium dark:text-slate-100 truncate">{{ a.nama }}</p>
                <p class="text-xs text-gray-500 dark:text-slate-400 mt-0.5">{{ a.jabatan || '-' }} · {{ a.divisi || '-' }}</p>
                <span class="inline-block mt-2 px-2 py-1 rounded-full text-xs"
                  :class="a.status_keanggotaan === 'aktif'
                    ? 'bg-green-100 text-green-700 dark:bg-emerald-900/40 dark:text-emerald-300'
                    : 'bg-gray-200 text-gray-600 dark:bg-slate-700 dark:text-slate-400'">
                  {{ a.status_keanggotaan }}
                </span>
              </div>
              <qrcode-vue :value="a.nomor_anggota || String(a.id)" :size="44" class="flex-shrink-0" />
            </div>
          </div>
          <p v-if="!anggotaIntern.length" class="text-sm text-gray-400 dark:text-slate-500 text-center py-6">
            Belum ada data.
          </p>
        </div>
      </div>
    </div>

    <!-- ===== AKUN EXTERN (semua role login bisa lihat) ===== -->
    <div>
      <h1 class="text-xl sm:text-2xl font-bold mb-4 dark:text-slate-100">
        {{ isSuperAdmin ? 'Akun Extern' : 'Anggota' }}
      </h1>

      <div v-if="loadingExtern" class="text-sm text-gray-400 dark:text-slate-500 py-4">Memuat...</div>
      <div v-else-if="errorExtern" class="text-sm text-rose-600 dark:text-rose-400 py-4">{{ errorExtern }}</div>

      <div v-else>
        <!-- Tampilan tabel (desktop/tablet ke atas) -->
        <div class="hidden sm:block bg-white dark:bg-slate-800 rounded-xl border border-gray-200 dark:border-slate-700 overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="bg-gray-50 dark:bg-slate-700/50">
              <tr class="text-left text-gray-500 dark:text-slate-400">
                <th class="p-3">Nama</th><th>RT</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="a in anggotaExtern" :key="a.id" class="border-t border-gray-100 dark:border-slate-700 dark:text-slate-200">
                <td class="p-3">{{ a.nama }}</td>
                <td>{{ a.rt || '-' }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Tampilan kartu (mobile) -->
        <div class="sm:hidden space-y-2">
          <div v-for="a in anggotaExtern" :key="a.id"
            class="bg-white dark:bg-slate-800 border border-gray-200 dark:border-slate-700 rounded-xl p-3 flex justify-between items-center">
            <span class="font-medium dark:text-slate-100">{{ a.nama }}</span>
            <span class="text-xs text-gray-500 dark:text-slate-400">RT {{ a.rt || '-' }}</span>
          </div>
          <p v-if="!anggotaExtern.length" class="text-sm text-gray-400 dark:text-slate-500 text-center py-6">
            Belum ada data.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import QrcodeVue from 'qrcode.vue'
import api from '../api/axios'
import { useAuthStore } from '../stores/auth'

const auth = useAuthStore()

const isSuperAdmin = computed(() => {
  try {
    return auth.hasRole('Super Admin')
  } catch {
    return false
  }
})

const anggotaIntern = ref([])
const anggotaExtern = ref([])
const loadingIntern = ref(false)
const loadingExtern = ref(false)
const errorExtern = ref('')

async function loadIntern() {
  if (!isSuperAdmin.value) return
  loadingIntern.value = true
  try {
    const { data } = await api.get('/anggota')
    anggotaIntern.value = data.data
  } catch (e) {
    console.error('Gagal memuat Akun Intern:', e)
  } finally {
    loadingIntern.value = false
  }
}

async function loadExtern() {
  loadingExtern.value = true
  errorExtern.value = ''
  try {
    const { data } = await api.get('/anggota-iuran')
    anggotaExtern.value = data
  } catch (e) {
    errorExtern.value = 'Gagal memuat daftar anggota.'
    console.error('Gagal memuat Akun Extern:', e)
  } finally {
    loadingExtern.value = false
  }
}

onMounted(() => {
  loadIntern()
  loadExtern()
})
</script>