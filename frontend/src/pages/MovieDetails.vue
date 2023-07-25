<template>
  <div v-if="!movieResource.loading && movieResource.doc">
    <h1 class="text-gray-900 font-bold text-[32px]">{{ movieDoc.title }}</h1>
    <div class="mt-11 flex flex-row items-center justify-between">
      <div class="flex flex-col space-y-3">
        <h2 class="text-gray-700 text-base font-bold uppercase">Director</h2>
        <p class="text-gray-600 text-xl font-semibold">
          {{ movieDoc.director }}
        </p>
      </div>

      <div class="flex flex-col space-y-3">
        <h2 class="text-gray-700 text-base font-bold uppercase">
          Release Date
        </h2>
        <p class="text-gray-600 text-xl font-semibold">
          {{ movieDoc.release_date }}
        </p>
      </div>
    </div>

    <div class="max-w-full">
      <div class="mx-12" v-if="currentStep === 0">
        <div class="p-2 mt-7 bg-white shadow-2xl rounded">
          <img :src="movieDoc.poster" alt="Movie Poster" />
        </div>

        <div class="w-full flex items-center justify-center mt-7">
          <Button size="xl" variant="solid" @click="currentStep++"
            >Book Tickets</Button
          >
        </div>

        <div class="flex flex-col space-y-3 mt-16">
          <h2 class="text-gray-700 text-base font-bold uppercase">Synopsis</h2>
          <p class="text-gray-600 text-lg font-normal">
            {{ movieDoc.synopsis }}
          </p>
        </div>
      </div>

      <div v-else-if="currentStep === 1">
        <h2 class="font-medium text-xl mt-7 text-gray-900">How many seats?</h2>

        <div class="flex flex-col w-full space-y-5 mt-6">
          <Button
            size="lg"
            :variant="index === bookingData.numberOfSeats ? 'subtle' : 'white'"
            class="shadow-lg"
            v-for="index in 8"
            @click="setNumberOfSeats(index)"
            >{{ index }}</Button
          >
        </div>
      </div>

      <div v-else-if="currentStep === 2">
        <div class="flex flex-col space-y-4">
          <h2 class="font-medium text-xl mt-7 text-gray-900">Date</h2>
          <Input type="date" v-model="bookingData.date" />
        </div>

        <div class="flex flex-col space-y-4">
          <h2 class="font-medium text-xl mt-7 text-gray-900">Cinema & Show</h2>

          <div class="space-y-2">
            <div
              v-for="theater in Object.keys(theaterShows.data)"
              :key="theater.name"
              class="bg-white shadow-xl p-4 rounded flex flex-col space-y-4"
            >
              <h3 class="text-sm font-bold text-gray-800">{{ theater }}</h3>
              <div class="flex flex-row space-x-2">
                <Button
                  @click="bookingData.show = show.name"
                  :key="show.name"
                  v-for="show in theaterShows.data[theater]"
                  size="sm"
                  :variant="
                    show.name === bookingData.show ? 'subtle' : 'outline'
                  "
                  >{{ show.start_time }}</Button
                >
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="currentStep === 3">
        <h2 class="font-medium text-xl mt-7 text-gray-900">Select Seats</h2>

        <div>
          <div
            :key="row"
            class="flex flex-row"
            v-for="row in Object.keys(seatStructure)"
          >
            <span
              @click="selectSeat(row, seat[0])"
              v-for="seat in seatStructure[row]"
              class="w-6 h-8 m-2 rounded-[2px]"
              :class="[
                seat[1] === 'Available'
                  ? 'bg-blue-300'
                  : seat[1] === 'Selected'
                  ? 'bg-blue-600'
                  : 'bg-gray-300',
                hasSelectedCorrectNumberOfSeats
                  ? 'cursor-not-allowed'
                  : 'cursor-pointer',
              ]"
            ></span>
          </div>
        </div>
      </div>

      <div v-else-if="currentStep === 4">
        <div class="w-full flex items-center flex-col mt-7">
          <h1 class="text-[110px]">🍿</h1>
          <h2 class="font-medium text-xl mt-7 text-gray-900">
            Enjoy the movie!
          </h2>
        </div>
      </div>
    </div>

    <div class="flex flex-row mt-6 space-x-2">
      <Button
        size="lg"
        variant="subtle"
        v-if="currentStep !== 0 && currentStep !== 4"
        @click="currentStep--"
        >Go Back</Button
      >

      <Button
        v-if="currentStep !== 0 && currentStep !== 4"
        :disabled="!nextButtonEnabled"
        size="lg"
        variant="solid"
        @click="handleNextClick()"
        >Next</Button
      >
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { createDocumentResource, createListResource } from 'frappe-ui'

const props = defineProps({
  movieName: {
    type: String,
    required: true,
  },
})

const movieResource = createDocumentResource({
  doctype: 'Movie',
  name: props.movieName,
  onSuccess(doc) {
    console.log(doc)
  },
  auto: true,
})

const theaterShows = createListResource({
  doctype: 'Theater Show',
  fields: ['theater', 'start_time', 'name'],
  auto: true,
  transform: (shows) => {
    const groupedShows = {}
    for (const show of shows) {
      if (!groupedShows[show.theater]) {
        groupedShows[show.theater] = []
      }
      groupedShows[show.theater].push(show)
    }

    return groupedShows
  },
})

const movieBooking = createListResource({ doctype: 'Ticket Booking', insert: {
  onSuccess() {
    console.log("Booking Created!")
    currentStep.value++
  }
} })

function getSeatStructure(alphabets, numbers) {
  const structure = {}
  for (const alphabet of alphabets) {
    structure[alphabet] = []
    for (const number of numbers) {
      structure[alphabet].push([number, 'Available'])
    }
  }
  return structure
}

const seatStructure = reactive(
  getSeatStructure(['A', 'B', 'C', 'D', 'E'], [1, 2, 3, 4, 5, 6, 7])
)

const today = new Date().toISOString().substr(0, 10)
const currentStep = ref(0)
const bookingData = reactive({
  numberOfSeats: 1,
  selectedSeats: [],
  date: today,
  show: null,
})

function setNumberOfSeats(n) {
  bookingData.numberOfSeats = n
}

function selectSeat(row, number) {
  // select exactly the number of seats
  if (hasSelectedCorrectNumberOfSeats.value) {
    return
  }

  // mark the seat booked in structure
  const seat = seatStructure[row].find((seat) => seat[0] === number)
  seat[1] = 'Selected'
  bookingData.selectedSeats.push(`${row}${number}`)
}

const hasSelectedCorrectNumberOfSeats = computed(() => {
  return bookingData.selectedSeats.length === bookingData.numberOfSeats
})

const movieDoc = computed(() => movieResource.doc)

const nextButtonEnabled = computed(() => {
  if (currentStep.value === 1) {
    return bookingData.numberOfSeats
  } else if (currentStep.value === 2) {
    return bookingData.date && bookingData.show
  } else if (currentStep.value === 3) {
    return hasSelectedCorrectNumberOfSeats.value
  }
})

function handleNextClick() {
  if (currentStep.value != 3) {
    currentStep.value++
    return
  }

  // create booking
  movieBooking.insert.submit({
    movie: props.movieName,
    date: bookingData.date,
    show: bookingData.show,
    selected_seats: JSON.stringify(bookingData.selectedSeats),
    number_of_tickets: bookingData.numberOfSeats,
  })
}
</script>
