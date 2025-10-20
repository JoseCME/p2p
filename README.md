# Pomodoro Timer App

Una aplicación de temporizador Pomodoro elegante y funcional construida con Vue.js, diseñada para mejorar la productividad mediante la técnica de trabajo por intervalos. Combina un diseño visual atractivo con funcionalidades avanzadas como notificaciones, sonidos personalizables y integración con tareas.

## Características Principales

- **Temporizador Personalizable**: Configura tiempos de trabajo (15-60 min), descanso corto (5-20 min) y descanso largo (15-30 min).
- **Animaciones Visuales**: El tomate se "llena" con el progreso del tiempo, acompañado de partículas flotantes para un efecto dinámico.
- **Notificaciones y Sonidos**: Alertas nativas del navegador con sonidos invasivos para asegurar que no te pierdas ningún cambio de sesión. Incluye vibración en dispositivos móviles.
- **Edición en Tiempo Real**: Haz doble clic en el tiempo para editarlo manualmente.
- **Integración con Tareas**: Completa tareas directamente desde el temporizador si hay una tarea seleccionada.
- **Persistencia de Configuración**: Guarda automáticamente tus preferencias en localStorage.
- **Interfaz Intuitiva**: Diseño moderno con gradientes, sombras y transiciones suaves, optimizado para una experiencia inmersiva.
- **Modo Oscuro Implícito**: Fondo borroso y colores vibrantes para una apariencia profesional.

## Tecnologías Utilizadas

- **Vue 3**: Framework principal con Composition API.
- **Web Audio API**: Para generar sonidos personalizados sin archivos externos.
- **Notifications API**: Para alertas del sistema.
- **Vibration API**: Para retroalimentación háptica en móviles.
- **CSS Animations**: Para efectos visuales fluidos.

## Instalación

1. Clona el repositorio:
   ```
   git clone https://github.com/tu-usuario/pomodoro-timer-app.git
   cd pomodoro-timer-app
   ```

2. Instala las dependencias:
   ```
   npm install
   ```

3. Ejecuta la aplicación en modo desarrollo:
   ```
   npm run dev
   ```

4. Abre tu navegador en `http://localhost:3000` (o el puerto configurado).

## Uso

1. **Configuración**: Ajusta los tiempos de trabajo y descanso en la sección de configuración antes de iniciar.
2. **Iniciar/Pausar**: Usa el botón "Iniciar" para comenzar el temporizador. El tomate se animará con el progreso.
3. **Cambios Automáticos**: Al finalizar un pomodoro de trabajo, automáticamente pasa a descanso corto o largo (cada 4 pomodoros).
4. **Notificaciones**: Recibe alertas cuando cambie de sesión, con sonidos y vibraciones.
5. **Completar Tareas**: Si tienes una tarea seleccionada (integrada desde otro componente), usa el botón para marcarla como completada.
6. **Reiniciar**: Presiona "Reiniciar" para volver al estado inicial.

## Contribución

¡Las contribuciones son bienvenidas! Abre un issue o envía un pull request para mejoras, como nuevos temas, idiomas adicionales o integraciones con otras apps de productividad.

## Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo LICENSE para más detalles.

---

¡Mejora tu enfoque y productividad con esta herramienta Pomodoro visualmente atractiva y funcional!
