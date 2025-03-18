<template>
  <div class="content">
    <h3>Todas las Citas</h3>
    <div v-if="appointments.length > 0">
      <ul>
        <li v-for="appointment in appointments" :key="appointment.id">
          <p><strong>Centro:</strong> {{ appointment.center }}</p>
          <p><strong>Fecha y Hora:</strong> {{ formatAppointmentDate(appointment) }}</p>
        </li>
      </ul>
      <!-- Agregar el gráfico aquí -->
      <h4>Gráfico de Citas por Centro</h4>
      <BarChart :data="chartData" />
    </div>
    <p v-else>No hay citas disponibles.</p>

    <!-- Botón para volver atrás -->
    <button @click="goBack" class="back-button">Volver atrás</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { format } from 'date-fns';
import { useRouter } from 'vue-router'; // Importar useRouter
import { apiService } from '@/services/apiService'; // Asegúrate de que esta ruta esté correcta
import { Bar } from 'vue-chartjs';
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';

// Registra los componentes necesarios de Chart.js
ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);

const appointments = ref([]);
const chartData = ref({
  labels: [],
  datasets: [
    {
      label: 'Citas por Centro',
      data: [],
      backgroundColor: 'rgba(75, 192, 192, 0.2)',
      borderColor: 'rgba(75, 192, 192, 1)',
      borderWidth: 1,
    },
  ],
});

// Obtener todas las citas de la API
const fetchAppointments = async () => {
  try {
    const data = await apiService.getAppointments(); // Usamos el método correcto de la API
    console.log("Citas obtenidas:", data); // Verificar datos en la consola
    appointments.value = data;

    // Actualizar los datos para el gráfico
    updateChartData(data);
  } catch (error) {
    console.error('Error al obtener citas:', error);
  }
};

// Actualizar los datos del gráfico
const updateChartData = (appointments) => {
  const centers = {};

  // Contamos las citas por centro
  appointments.forEach((appointment) => {
    const center = appointment.center;
    centers[center] = centers[center] ? centers[center] + 1 : 1;
  });

  // Actualizamos las etiquetas y los datos del gráfico
  chartData.value.labels = Object.keys(centers);
  chartData.value.datasets[0].data = Object.values(centers);
};

// Formatear la fecha y hora
const formatAppointmentDate = (appointment) => {
  if (appointment.fecha && appointment.hora) {
    // Asegúrate de que 'fecha' y 'hora' son los campos correctos
    return format(new Date(`${appointment.fecha}T${appointment.hora}`), 'dd/MM/yyyy HH:mm');
  }
  return 'Fecha no disponible'; // Si no hay fecha o hora
};

// Obtener citas cuando el componente se monta
onMounted(() => {
  fetchAppointments();
});

// Función para navegar atrás
const router = useRouter();
const goBack = () => {
  router.back(); // Esto va a llevar al usuario a la página anterior
};
</script>

<script>
export default {
  components: {
    BarChart: Bar, // Agregar el componente de gráfico de barras
  },
};
</script>

<style scoped>
@import url(../assets/TodasLasCitasStyles.scss);
</style>
