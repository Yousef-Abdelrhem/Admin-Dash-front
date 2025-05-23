<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import axios from "axios";
import AddImage from "../components/AddImage.vue";
import Avatar from "../components/Avatar.vue";

const route = useRoute();
const productId = route.params.id as string | undefined; // gets the id from /editProduct/:id

const name = ref("222");
const description = ref("");
const price = ref("");
const inStock = ref("");
const category = ref("");
const images = ref<File[]>([]);
const error = ref<string | null>(null);

// Handle image input
function handleImageUpload(file: File, index: number) {
  images.value[index] = file;
  console.log(images.value);
}

// Load existing product data if editing
onMounted(async () => {
  const token = localStorage.getItem("token");

  if (productId) {
    try {
      const response = await axios.get(
        `https://admin-dashboard-gilt-omega.vercel.app/api/products/${productId}`,
        {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        },
      );
      const product = response.data;
      name.value = product.name;
      description.value = product.description;
      price.value = product.price;
      inStock.value = product.inStock;
      category.value = product.category;
    } catch (err) {
      console.error("Failed to load product data", err);
    }
  }
});

// Submit form handler
async function submitProduct() {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      alert("Authorization token missing. Please log in.");
      return;
    }

    const formData = new FormData();
    formData.append("name", name.value);
    formData.append("description", description.value);
    formData.append("price", price.value);
    formData.append("inStock", inStock.value);
    formData.append("category", category.value);
    // if (productId) formData.append('productId', productId);

    formData.forEach((key) => {
      console.log(key);
    });

    const url = productId
      ? `https://admin-dashboard-gilt-omega.vercel.app/api/products/${productId}`
      : `https://admin-dashboard-gilt-omega.vercel.app/api/products/`;

    const method = productId ? "patch" : "post";

    await axios({
      method,
      url,
      data: formData,
      headers: {
        "Content-Type": "multipart/form-data",
        Authorization: `Bearer ${token}`, // 🔐 Add token here
      },
    });

    alert(`Product ${productId ? "updated" : "added"} successfully!`);
  } catch (err: any) {
    alert(err.response?.data?.message || "Failed to submit product.");
  }
}
</script>

<template>
  <div class="mb-10 pr-2 pl-2">
    <div class="mt-2 mb-3 flex justify-end gap-1 lg:mt-10 lg:gap-10">
      <Avatar />
    </div>
    <section
      class="w-full rounded-xl border-2 border-[#EFEFEF] p-4 pt-5 pr-4 pb-5"
    >
      <h2 class="text-main-950 mb-6 text-2xl font-bold">Edit Product</h2>

      <div class="flex flex-col gap-4 md:flex-row">
        <div
          class="flex flex-row flex-wrap justify-center gap-3 md:flex-col md:justify-start md:gap-5"
        >
          <AddImage @upload="(file) => handleImageUpload(file, 0)" />
          <AddImage @upload="(file) => handleImageUpload(file, 1)" />
          <AddImage @upload="(file) => handleImageUpload(file, 2)" />
          <AddImage @upload="(file) => handleImageUpload(file, 3)" />
        </div>

        <form
          @submit.prevent="submitProduct"
          class="mb-2 flex w-full flex-col justify-center gap-6 rounded-xl border-2 border-[#B0B0B0] p-4 pt-5 pr-4 pb-5"
        >
          <fieldset class="fieldset flex">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Product Name
            </legend>
            <input
              v-model="name"
              type="text"
              class="input w-full focus-within:border-0"
              placeholder="bags"
            />
          </fieldset>

          <fieldset class="fieldset">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Product Description
            </legend>
            <textarea
              v-model="description"
              class="textarea h-[10rem] w-full resize-none p-2 text-black outline-none focus-within:border-0"
              placeholder="Small Bag"
            ></textarea>
          </fieldset>

          <fieldset class="fieldset flex">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Product Category
            </legend>
            <select
              v-model="category"
              class="select select-lg w-full bg-[url('data:image/svg+xml;charset=UTF-8,%3Csvg%20width%3D%2215%22%20height%3D%2211%22%20viewBox%3D%220%200%2015%2011%22%20fill%3D%22none%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%3E%3Cpath%20d%3D%22M1.50006%200.333324H13.5001C13.6216%200.333705%2013.7406%200.367214%2013.8445%200.430243C13.9484%200.493272%2014.0331%200.583434%2014.0895%200.691026C14.146%200.798617%2014.172%200.919564%2014.1648%201.04085C14.1576%201.16213%2014.1175%201.27915%2014.0487%201.37932L8.04872%2010.046C7.80006%2010.4053%207.20139%2010.4053%206.95206%2010.046L0.952057%201.37932C0.882607%201.27936%200.84188%201.16228%200.8343%201.04079C0.826721%200.919311%200.85258%200.798072%200.909066%200.690252C0.965553%200.582433%201.05051%200.492155%201.1547%200.429228C1.25889%200.366302%201.37834%200.333133%201.50006%200.333324Z%22%20fill%3D%22%23763A26%22%2F%3E%3C%2Fsvg%3E')] bg-[length:15px] bg-[right_1rem_center] bg-no-repeat pr-10 text-[0.9rem] focus-within:border-0"
            >
              <option disabled value="">Large</option>
              <option>Large Apple</option>
              <option>Large Orange</option>
              <option>Large Tomato</option>
            </select>
          </fieldset>

          <fieldset class="fieldset flex">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Product Price
            </legend>
            <input
              v-model="price"
              type="text"
              class="input w-full focus-within:border-0"
              placeholder="$200"
            />
          </fieldset>

          <fieldset class="fieldset flex">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Product Discount
            </legend>
            <input
              type="text"
              class="input w-full focus-within:border-0"
              placeholder="20%"
            />
          </fieldset>

          <fieldset class="fieldset flex">
            <legend class="fieldset-legend font--light mb-[0.5rem]">
              Quantity In Stock
            </legend>
            <input
              v-model="inStock"
              type="text"
              class="input w-full focus-within:border-0"
              placeholder="200"
            />
          </fieldset>

          <button
            type="submit"
            class="bg-main-200 hover:bg-main-300/70 flex w-full items-center justify-center gap-2 self-center rounded-xl py-3 md:w-[10rem]"
          >
            <svg
              width="18"
              height="18"
              viewBox="0 0 18 18"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M2.25 13.4856V9.53063C2.25 6.80562 2.25 5.44311 3.07376 4.59655C3.89753 3.75 5.22335 3.75 7.875 3.75C10.5266 3.75 11.8525 3.75 12.6763 4.59655C13.5 5.44311 13.5 6.80562 13.5 9.53063V13.4856C13.5 15.215 13.5 16.0797 12.9566 16.3892C11.9043 16.9886 9.93037 14.9889 8.99295 14.3868C8.44927 14.0376 8.17747 13.863 7.875 13.863C7.57253 13.863 7.30069 14.0376 6.75704 14.3868C5.81963 14.9889 3.84572 16.9886 2.7934 16.3892C2.25 16.0797 2.25 15.215 2.25 13.4856Z"
                stroke="#763A26"
                stroke-width="1.125"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path
                d="M6.75 1.5H8.25C11.7855 1.5 13.5533 1.5 14.6516 2.59835C15.75 3.6967 15.75 5.46446 15.75 9V13.5"
                stroke="#763A26"
                stroke-width="1.125"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>

            <span class="text-main-950 text-lg font-medium">Save</span>
          </button>
        </form>
      </div>
    </section>
  </div>
</template>
