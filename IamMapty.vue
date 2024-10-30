<template>
    <div>
      <div class="sidebar">
        <img src="logo.png" alt="Logo" class="logo" />
  
        <ul class="workouts">
          <form @submit.prevent="newWorkout" class="form" :class="{ hidden: isFormHidden }">
            <div class="form__row">
              <label class="form__label">Type</label>
              <select v-model="workoutType" class="form__input form__input--type">
                <option value="running">Running</option>
                <option value="cycling">Cycling</option>
              </select>
            </div>
            <div class="form__row">
              <label class="form__label">Distance</label>
              <input v-model.number="distance" class="form__input form__input--distance" placeholder="km" />
            </div>
            <div class="form__row">
              <label class="form__label">Duration</label>
              <input v-model.number="duration" class="form__input form__input--duration" placeholder="min" />
            </div>
  
            <div class="form__row">
              <label class="form__label">Cadence</label>
              <input v-model.number="cadence" class="form__input form__input--cadence" placeholder="step/min" />
            </div>
            <div class="form__row" :class="{ 'form__row--hidden': workoutType === 'cycling' }">
              <label class="form__label">Elev Gain</label>
              <input v-model.number="elevation" class="form__input form__input--elevation" placeholder="meters" />
            </div>
            <button class="form__btn">OK</button>
          </form>
  
          <li v-for="workout in workouts" :key="workout.id" :class="`workout workout--${workout.type}`" :data-id="workout.id" @click="moveToPopup(workout)">
            <h2 class="workout__title">{{ workout.description }}</h2>
            <div class="workout__details">
              <span class="workout__icon">{{ workout.type === 'running' ? '🏃‍♂️' : '🚴‍♀️' }}</span>
              <span class="workout__value">{{ workout.distance }}</span>
              <span class="workout__unit">km</span>
            </div>
            <div class="workout__details">
              <span class="workout__icon">⏱</span>
              <span class="workout__value">{{ workout.duration }}</span>
              <span class="workout__unit">min</span>
            </div>
            <div class="workout__details" v-if="workout.type === 'running'">
              <span class="workout__icon">⚡️</span>
              <span class="workout__value">{{ workout.pace.toFixed(1) }}</span>
              <span class="workout__unit">min/km</span>
            </div>
            <div class="workout__details" v-if="workout.type === 'running'">
              <span class="workout__icon">🦶🏼</span>
              <span class="workout__value">{{ workout.cadence }}</span>
              <span class="workout__unit">spm</span>
            </div>
            <div class="workout__details" v-if="workout.type === 'cycling'">
              <span class="workout__icon">⚡️</span>
              <span class="workout__value">{{ workout.speed.toFixed(1) }}</span>
              <span class="workout__unit">km/h</span>
            </div>
            <div class="workout__details" v-if="workout.type === 'cycling'">
              <span class="workout__icon">⛰</span>
              <span class="workout__value">{{ workout.elevationGain }}</span>
              <span class="workout__unit">m</span>
            </div>
          </li>
        </ul>
  
        <p class="copyright">
          &copy; Copyright by
          <a class="twitter-link" target="_blank" href="https://twitter.com/jonasschmedtman">Jonas Schmedtmann</a>.
          Use for learning or your portfolio. Don't use to teach. Don't claim as your own.
        </p>
      </div>
  
      <div id="map"></div>
    </div>
  </template>
  
  <script>
  import L from 'leaflet';
  
  class Workout {
    constructor(coords, distance, duration, extra, type) {
      this.coords = coords; // [lat, lng]
      this.distance = distance; // in km
      this.duration = duration; // in min
      this.type = type;
  
      if (type === 'running') {
        this.cadence = extra;
        this.calcPace();
      }
  
      if (type === 'cycling') {
        this.elevationGain = extra;
        this.calcSpeed();
      }
  
      this._setDescription();
      this.id = (Date.now() + '').slice(-10);
      this.clicks = 0;
    }
  
    _setDescription() {
      const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
      this.description = `${this.type[0].toUpperCase()}${this.type.slice(1)} on ${months[new Date().getMonth()]} ${new Date().getDate()}`;
    }
  
    calcPace() {
      this.pace = this.duration / this.distance; // min/km
    }
  
    calcSpeed() {
      this.speed = this.distance / (this.duration / 60); // km/h
    }
  
    click() {
      this.clicks++;
    }
  }
  
  export default {
    data() {
      return {
        workouts: [],
        workoutType: 'running',
        distance: 0,
        duration: 0,
        cadence: 0,
        elevation: 0,
        isFormHidden: true,
        map: null,
        mapZoomLevel: 13,
        mapEvent: null,
      };
    },
  
    mounted() {
      this.getPosition();
      this.getLocalStorage();
    },
  
    methods: {
      getPosition() {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(this.loadMap, () => {
            alert('Could not get your position');
          });
        }
      },
  
      loadMap(position) {
        const { latitude, longitude } = position.coords;
        const coords = [latitude, longitude];
  
        this.map = L.map('map').setView(coords, this.mapZoomLevel);
        L.tileLayer('https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png', {
          attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        }).addTo(this.map);
  
        this.map.on('click', this.showForm);
        this.workouts.forEach(work => {
          this.renderWorkoutMarker(work);
        });
      },
  
      showForm(mapE) {
        this.mapEvent = mapE;
        this.isFormHidden = false;
      },
  
      hideForm() {
        this.isFormHidden = true;
        this.resetForm();
      },
  
      resetForm() {
        this.distance = 0;
        this.duration = 0;
        this.cadence = 0;
        this.elevation = 0;
      },
  
      newWorkout() {
        const validInputs = (...inputs) => inputs.every(inp => Number.isFinite(inp));
        const allPositive = (...inputs) => inputs.every(inp => inp > 0);
  
        const type = this.workoutType;
        const distance = +this.distance;
        const duration = +this.duration;
        const { lat, lng } = this.mapEvent.latlng;
        let workout;
  
        if (type === 'running') {
          const cadence = +this.cadence;
          if (!validInputs(distance, duration, cadence) || !allPositive(distance, duration, cadence))
            return alert('Inputs have to be positive numbers!');
          workout = new Workout([lat, lng], distance, duration, cadence, 'running');
        }
  
        if (type === 'cycling') {
          const elevation = +this.elevation;
          if (!validInputs(distance, duration, elevation) || !allPositive(distance, duration))
            return alert('Inputs have to be positive numbers!');
          workout = new Workout([lat, lng], distance, duration, elevation, 'cycling');
        }
  
        this.workouts.push(workout);
        this.renderWorkoutMarker(workout);
        this.renderWorkout(workout);
        this.hideForm();
        this.setLocalStorage();
      },
  
      renderWorkoutMarker(workout) {
        L.marker(workout.coords)
          .addTo(this.map)
          .bindPopup(
            L.popup({
              maxWidth: 250,
              minWidth: 100,
              autoClose: false,
              closeOnClick: false,
              className: `${workout.type}-popup`,
            })
          )
          .setPopupContent(`${workout.type === 'running' ? '🏃‍♂️' : '🚴‍♀️'} ${workout.description}`)
          .openPopup();
      },
  
      renderWorkout(workout) {
        this.workouts.push(workout);
      },
  
      moveToPopup(workout) {
        if (!this.map) return;
  
        this.map.setView(workout.coords, this.mapZoomLevel, {
          animate: true,
          pan: { duration: 1 },
        });
        workout.click();
      },
  
      setLocalStorage() {
        localStorage.setItem('workouts', JSON.stringify(this.workouts));
      },
  
      getLocalStorage() {
        const data = JSON.parse(localStorage.getItem('workouts'));
        if (!data) return;
  
        this.workouts = data.map(work => new Workout(work.coords, work.distance, work.duration, work.type === 'running' ? work.cadence : work.elevationGain, work.type));
        this.workouts.forEach(work => {
          this.renderWorkoutMarker(work);
        });
      },
  
      reset() {
        localStorage.removeItem('workouts');
        location.reload();
      },
    },
  };
  </script>
  
  <style>
  /* Add your CSS styling here */
  </style>
  