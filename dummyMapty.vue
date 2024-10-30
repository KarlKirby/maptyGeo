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
  
          <li v-for="workout in workouts" :key="workout.id" :class="`workout workout--${workout.type}`" :data-id="workout.id">
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
  
      <div id=" map"> <!-- This will be replaced with a map later -->
        <p>Dummy map area</p>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        workouts: [
          {
            id: 1,
            type: 'running',
            distance: 5.2,
            duration: 24,
            cadence: 178,
            pace: 4.6,
            description: 'Running on April 14',
            coords: [39, -12],
          },
          {
            id: 2,
            type: 'cycling',
            distance: 27,
            duration: 95,
            elevationGain: 223,
            speed: 16,
            description: 'Cycling on April 5',
            coords: [40, -13],
          },
        ],
        workoutType: 'running',
        distance: 0,
        duration: 0,
        cadence: 0,
        elevation: 0,
        isFormHidden: true,
      };
    },
  
    methods: {
      newWorkout(e) {
        e.preventDefault();
  
        const validInputs = (...inputs) => inputs.every(inp => Number.isFinite(inp));
        const allPositive = (...inputs) => inputs.every(inp => inp > 0);
  
        const type = this.workoutType;
        const distance = +this.distance;
        const duration = +this.duration;
        let workout;
  
        if (type === 'running') {
          const cadence = +this.cadence;
  
          if (!validInputs(distance, duration, cadence) || !allPositive(distance, duration, cadence))
            return alert('Inputs have to be positive numbers!');
  
          workout = {
            id: Date.now(),
            type,
            distance,
            duration,
            cadence,
            pace: duration / distance,
            description: `${type[0].toUpperCase()}${type.slice(1)} on ${new Date().toLocaleDateString()}`,
            coords: [Math.random() * 100, Math.random() * 100],
          };
        }
  
        if (type === 'cycling') {
          const elevation = +this.elevation;
  
          if (!validInputs(distance, duration, elevation) || !allPositive(distance, duration))
            return alert('Inputs have to be positive numbers!');
  
          workout = {
            id: Date.now(),
            type,
            distance,
            duration,
            elevationGain: elevation,
            speed: distance / (duration / 60),
            description: `${type[0].toUpperCase()}${type.slice(1)} on ${new Date().toLocaleDateString()}`,
            coords: [Math.random() * 100, Math.random() * 100],
          };
        }
  
        this.workouts.push(workout);
  
        this.distance = 0;
        this.duration = 0;
        this.cadence = 0;
        this.elevation = 0;
      },
    },
  };
  </script>
  
  <style>
  /* Add your CSS styles here */
  </style>