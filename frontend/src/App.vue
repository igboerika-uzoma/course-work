

<template>
  <div id="app">
    <header>
      <div class="header-container">
        <div class="logo">
          <i class="fas fa-book"></i>
          AfterClass
        </div>
        <nav>
          <a href="#lessons">Lessons</a>
          <a href="#about">About</a>
          <a href="#contact">Contact</a>
          <div class="cart-icon" @click="showCart">
            <i class="fas fa-shopping-cart"></i>
            <span v-if="cartItemCount > 0" class="cart-badge">{{ cartItemCount }}</span>
          </div>
        </nav>
      </div>
    </header>

    <!-- Main Content -->
    <div class="container">
      <!-- Lessons Section -->
      <div v-if="currentView === 'lessons'" id="lessonsSection">
        <h1 class="section-title">Available Classes</h1>

        <!-- Controls -->
        <div class="controls">
          <div class="control-group">
            <input 
              type="search" 
              v-model="searchTerm"
              placeholder="Search lessons..." 
              id="searchInput"
            >
          </div>
          <div class="control-group">
            <label for="sortSelect">Sort by:</label>
            <select id="sortSelect" v-model="sortBy">
              <option value="subject">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="spaces">Availability</option>
            </select>
          </div>
          <button class="btn btn-secondary" @click="toggleSortOrder">
            <i :class="sortAscending ? 'fas fa-arrow-up' : 'fas fa-arrow-down'"></i> 
            {{ sortAscending ? 'Ascending' : 'Descending' }}
          </button>
        </div>

        <!-- Lessons Grid using v-for -->
        <div class="lessons-grid">
          <div 
            v-for="lesson in filteredAndSortedLessons" 
            :key="lesson.id" 
            class="lesson-card"
          >
            <div class="lesson-image">
              <i :class="'fas ' + lesson.icon"></i>
            </div>
            <div class="lesson-content">
              <div class="lesson-subject">{{ lesson.subject }}</div>
              <div class="lesson-details">
                <div class="detail-item">
                  <i class="fas fa-map-marker-alt"></i>
                  <span>{{ lesson.location }}</span>
                </div>
                <div class="detail-item">
                  <i class="fas fa-pound-sign"></i>
                  <span>£{{ lesson.price }}</span>
                </div>
                <div class="detail-item">
                  <i class="fas fa-chair"></i>
                  <span>{{ lesson.spaces }} space{{ lesson.spaces !== 1 ? 's' : '' }} available</span>
                </div>
              </div>
              <button 
                class="btn btn-primary btn-block" 
                @click="addToCart(lesson.id)"
                :disabled="lesson.spaces === 0"
              >
                <i class="fas fa-plus"></i> 
                {{ lesson.spaces === 0 ? 'Sold Out' : 'Add to cart' }}
              </button>
            </div>
          </div>

          <!-- Empty State -->
          <div v-if="filteredAndSortedLessons.length === 0" class="empty-state">
            <div class="empty-state-icon">
              <i class="fas fa-inbox"></i>
            </div>
            <h3>No lessons found</h3>
            <p>Try adjusting your search or filters</p>
          </div>
        </div>
      </div>

      <!-- Cart Section -->
      <div v-if="currentView === 'cart'" id="cartSection">
        <h1 class="section-title">Shopping Cart</h1>
        <button class="btn btn-secondary" @click="showLessons" style="margin-bottom: 2rem;">
          <i class="fas fa-arrow-left"></i> Back to Lessons
        </button>

        <div v-if="orderSubmitted" class="success-message show">
          <i class="fas fa-check-circle"></i>
          <span>Order submitted successfully! We'll contact you soon.</span>
        </div>

        <div class="cart-container">
          <div class="cart-items">
            <!-- Empty Cart State -->
            <div v-if="cart.length === 0" class="empty-state">
              <div class="empty-state-icon">
                <i class="fas fa-shopping-cart"></i>
              </div>
              <h3>Your cart is empty</h3>
              <p>Add some lessons to get started!</p>
            </div>

            <!-- Cart Items using v-for -->
            <div 
              v-for="item in cart" 
              :key="item.id" 
              class="cart-item"
            >
              <div class="cart-item-info">
                <div class="cart-item-name">{{ item.subject }}</div>
                <div class="cart-item-location">{{ item.location }} • Quantity: {{ item.quantity }}</div>
              </div>
              <div class="cart-item-price">£{{ item.price * item.quantity }}</div>
              <button class="btn btn-danger btn-sm" @click="removeFromCart(item.id)">
                <i class="fas fa-trash"></i>
              </button>
            </div>
          </div>

          <div class="cart-summary">
            <div class="summary-title">Order Summary</div>
            <div class="summary-row">
              <span>Total Items:</span>
              <strong>{{ cartItemCount }}</strong>
            </div>
            <div class="summary-row">
              <span>Subtotal:</span>
              <strong>£{{ subtotal.toFixed(2) }}</strong>
            </div>
            <div class="summary-row">
              <span>Tax (10%):</span>
              <strong>£{{ tax.toFixed(2) }}</strong>
            </div>
            <div class="summary-total">
              Total: <span>£{{ total.toFixed(2) }}</span>
            </div>

            <form class="checkout-form" @submit.prevent="validateAndSubmitOrder" style="margin-top: 1.5rem; padding: 0; box-shadow: none; max-width: 100%;">
              <div class="form-group">
                <label for="nameInput">Full Name</label>
                <input 
                  type="text" 
                  id="nameInput"
                  v-model="customerName"
                  :class="{ error: nameError }"
                  placeholder="Enter your full name" 
                  required
                >
                <div class="error-message" :class="{ show: nameError }">
                  Please enter a valid name (letters only)
                </div>
              </div>

              <div class="form-group">
                <label for="phoneInput">Phone Number</label>
                <input 
                  type="tel" 
                  id="phoneInput"
                  v-model="customerPhone"
                  :class="{ error: phoneError }"
                  placeholder="Enter your phone number" 
                  required
                >
                <div class="error-message" :class="{ show: phoneError }">
                  Please enter a valid phone number (digits only)
                </div>
              </div>

              <button 
                type="submit" 
                class="btn btn-success btn-block"
                :disabled="cart.length === 0"
              >
                <i class="fas fa-credit-card"></i> Complete Checkout
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      // View state
      currentView: 'lessons', // 'lessons' or 'cart'
       apiURL: 'https://backend-express-app-yw9p.onrender.com',
      
      // Lessons data
      lessons: [
        { id: 1, subject: 'Math', location: 'London', price: 100, spaces: 5, icon: 'fa-calculator' },
        { id: 2, subject: 'English', location: 'Manchester', price: 85, spaces: 5, icon: 'fa-pen-fancy' },
        { id: 3, subject: 'Physics', location: 'Cambridge', price: 110, spaces: 5, icon: 'fa-atom' },
        { id: 4, subject: 'Chemistry', location: 'Oxford', price: 110, spaces: 5, icon: 'fa-flask' },
        { id: 5, subject: 'Biology', location: 'Edinburgh', price: 105, spaces: 5, icon: 'fa-microscope' },
        { id: 6, subject: 'Music', location: 'Bristol', price: 90, spaces: 5, icon: 'fa-music' },
        { id: 7, subject: 'Nutrition', location: 'Leeds', price: 75, spaces: 5, icon: 'fa-apple-alt' },
        { id: 8, subject: 'Literature', location: 'Glasgow', price: 80, spaces: 5, icon: 'fa-book' },
        { id: 9, subject: 'Calculus', location: 'Birmingham', price: 115, spaces: 5, icon: 'fa-square-root-alt' },
        { id: 10, subject: 'Geography', location: 'Liverpool', price: 95, spaces: 5, icon: 'fa-globe' }
      ],
      
      // Cart
      cart: [],
      
      // Sorting and filtering
      sortBy: 'subject',
      sortAscending: true,
      searchTerm: '',
      
      // Checkout form
      customerName: '',
      customerPhone: '',
      nameError: false,
      phoneError: false,
      orderSubmitted: false
    };
  },
  
  computed: {
    // Calculate total number of items in cart
    cartItemCount() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0);
    },
    
    // Calculate subtotal
    subtotal() {
      return this.cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    },
    
    // Calculate tax (10%)
    tax() {
      return this.subtotal * 0.1;
    },
    
    // Calculate total
    total() {
      return this.subtotal + this.tax;
    },
    
    // Filter and sort lessons
    filteredAndSortedLessons() {
      let filtered = this.lessons;
      
      // Apply search filter
      if (this.searchTerm) {
        const search = this.searchTerm.toLowerCase();
        filtered = filtered.filter(lesson => 
          lesson.subject.toLowerCase().includes(search) ||
          lesson.location.toLowerCase().includes(search) ||
          lesson.price.toString().includes(search) ||
          lesson.spaces.toString().includes(search)
        );
      }
      
      
      const sorted = [...filtered].sort((a, b) => {
        let aValue = a[this.sortBy];
        let bValue = b[this.sortBy];
        
        if (typeof aValue === 'string') {
          aValue = aValue.toLowerCase();
          bValue = bValue.toLowerCase();
        }
        
        if (aValue < bValue) return this.sortAscending ? -1 : 1;
        if (aValue > bValue) return this.sortAscending ? 1 : -1;
        return 0;
      });
      
      return sorted;
    }
  },
  
  methods: {
    addToCart(lessonId) {
       const lesson = this.lessons.find(l => l._id === lessonId);
      
      if (lesson && lesson.space > 0) {
        lesson.space--;
        
         const existingItem = this.cart.find(item => item._id === lessonId);
        
        if (existingItem) {
          existingItem.quantity++;
        } else {
          // Add new item to cart
          this.cart.push({
             _id: lesson._id,  
            subject: lesson.topic, 
            location: lesson.location,
            price: lesson.price,
            space: lesson.space,  
            quantity: 1
          });
        }
      }
    },
    
    // Remove lesson from cart
    removeFromCart(lessonId) {
      const cartItem = this.cart.find(item => item.id === lessonId);
      
      if (cartItem) {
        const lesson = this.lessons.find(l => l.id === lessonId);
        
        // Restore spaces
        lesson.spaces += cartItem.quantity;
        
        // Remove from cart
        this.cart = this.cart.filter(item => item.id !== lessonId);
      }
    },
    
    // Toggle sort order
    toggleSortOrder() {
      this.sortAscending = !this.sortAscending;
    },
    
    // Show cart view
    showCart() {
      this.currentView = 'cart';
      this.orderSubmitted = false;
    },
    
    // Show lessons view
    showLessons() {
      this.currentView = 'lessons';
      this.orderSubmitted = false;
    },
    
    // Validate and submit order
    validateAndSubmitOrder() {
      // Validate name (letters and spaces only)
      const nameValid = /^[a-zA-Z\s]+$/.test(this.customerName.trim());
      this.nameError = !nameValid;
      
      // Validate phone (digits only)
      const phoneValid = /^\d+$/.test(this.customerPhone.trim());
      this.phoneError = !phoneValid;
      
      if (nameValid && phoneValid) {
        // Create order object
        const order = {
          name: this.customerName,
          phone: this.customerPhone,
          lessonIds: this.cart.map(item => item.id),
          numberOfSpaces: this.cart.map(item => item.quantity),
          totalAmount: this.total
        };
        
        console.log('Order submitted:', order);
        
        // Show success message
        this.orderSubmitted = true;
        
        // Clear form
        this.customerName = '';
        this.customerPhone = '';
        this.nameError = false;
        this.phoneError = false;
        
        // Clear cart
        this.cart = [];
        
        // Return to lessons after 2 seconds
        setTimeout(() => {
          this.showLessons();
        }, 2000);
        
        // Server POST example commented out...
      }
    },
    async fetchLessons() {
      try {
        // ensure you have apiURL in data() or replace with a direct path
        const response = await fetch(`${this.apiURL}/lessons`);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        this.lessons = data;
        console.log('✅ Lessons loaded:', this.lessons.length);
      } catch (error) {
        console.error('❌ Error fetching lessons:', error);
        alert('Failed to load lessons. Please refresh the page.');
      }
    },

    async submitOrder(name, phone) {
            try {
                // Prepare order data from cart
                const lessonIDs = this.cart.map(item => item._id);
                const numberOfSpaces = this.cart.map(item => item.quantity);

                const orderData = {
                    name: name,
                    phone: phone,
                    lessonIDs: lessonIDs,
                    numberOfSpaces: numberOfSpaces
                };

                const response = await fetch(`${this.apiURL}/orders`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(orderData)
                });

                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const result = await response.json();
                console.log('✅ Order created:', result.orderId);
                
                // Update lesson spaces after order
                await this.updateLessonSpaces();
                
                return result;
                
            } catch (error) {
                console.error('❌ Error submitting order:', error);
                alert('Failed to submit order. Please try again.');
                throw error;
            }
        },

         async updateLessonSpaces() {
            try {
                // Update each lesson in the cart
                const updatePromises = this.cart.map(async (item) => {
                    const lessonId = item._id;
                    const newSpace = item.space - item.quantity;

                    const response = await fetch(`${this.apiURL}/lessons/${lessonId}`, {
                        method: 'PUT',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({ space: newSpace })
                    });

                    if (!response.ok) {
                        throw new Error(`Failed to update lesson ${lessonId}`);
                    }

                    return response.json();
                });

                await Promise.all(updatePromises);
                console.log('✅ All lesson spaces updated');
                
                // Refresh lessons from database
                await this.fetchLessons();
                
            } catch (error) {
                console.error('❌ Error updating lesson spaces:', error);
                throw error;
            }
        }
    },

    mounted() {
        this.fetchLessons();
    }
  }
  
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --primary-blue: #87CEEB;
  --secondary-blue: #4DA6D6;
  --dark-blue: #2E5C8A;
  --light-bg: #E0F6FF;
  --white: #FFFFFF;
  --text-dark: #1a1a1a;
  --text-light: #555555;
  --border-color: #B0E0E6;
  --success-green: #4CAF50;
  --danger-red: #FF6B6B;
  --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, var(--light-bg) 0%, #F0F9FF 100%);
  color: var(--text-dark);
  line-height: 1.6;
  min-height: 100vh;
}

header {
  background: linear-gradient(135deg, var(--secondary-blue) 0%, var(--dark-blue) 100%);
  color: var(--white);
  padding: 1rem 2rem;
  box-shadow: 0 4px 15px rgba(46, 92, 138, 0.2);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-container {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 2rem;
}

.logo {
  font-size: 1.8rem;
  font-weight: bold;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: var(--transition);
}

.logo:hover {
  transform: scale(1.05);
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

nav {
  display: flex;
  gap: 2rem;
  align-items: center;
}

nav a {
  color: var(--white);
  text-decoration: none;
  font-weight: 500;
  transition: var(--transition);
  position: relative;
}

nav a::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 0;
  width: 0;
  height: 2px;
  background: var(--primary-blue);
  transition: var(--transition);
}

nav a:hover::after {
  width: 100%;
}

.cart-icon {
  position: relative;
  cursor: pointer;
  font-size: 1.5rem;
}

.cart-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background: var(--danger-red);
  color: var(--white);
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  font-weight: bold;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

/* Main Container */
.container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

/* Section Title */
.section-title {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  text-align: center;
  color: var(--dark-blue);
  position: relative;
  padding-bottom: 1rem;
}

.section-title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100px;
  height: 4px;
  background: linear-gradient(90deg, var(--secondary-blue), var(--primary-blue));
  border-radius: 2px;
}

/* Controls Section */
.controls {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
  align-items: center;
}

.control-group {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex: 1;
  min-width: 200px;
}

.control-group label {
  font-weight: 600;
  color: var(--dark-blue);
}

/* Inputs & Selects */
input[type="text"],
input[type="search"],
input[type="tel"],
input[type="email"],
select {
  padding: 0.75rem 1rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  font-size: 1rem;
  transition: var(--transition);
  background: var(--white);
  color: var(--text-dark);
  flex: 1;
}

input[type="text"]:focus,
input[type="search"]:focus,
input[type="tel"]:focus,
input[type="email"]:focus,
select:focus {
  outline: none;
  border-color: var(--secondary-blue);
  box-shadow: 0 0 10px rgba(77, 166, 214, 0.3);
  transform: translateY(-2px);
}

input[type="text"]::placeholder,
input[type="search"]::placeholder {
  color: #aaa;
}

/* Buttons */
.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: var(--transition);
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  text-decoration: none;
  white-space: nowrap;
}

.btn-primary {
  background: linear-gradient(135deg, var(--primary-blue) 0%, var(--secondary-blue) 100%);
  color: var(--white);
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(77, 166, 214, 0.4);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(-1px);
}

.btn-secondary {
  background: var(--light-bg);
  color: var(--secondary-blue);
  border: 2px solid var(--secondary-blue);
}

.btn-secondary:hover:not(:disabled) {
  background: var(--secondary-blue);
  color: var(--white);
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(77, 166, 214, 0.4);
}

.btn-success {
  background: var(--success-green);
  color: var(--white);
}

.btn-success:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(76, 175, 80, 0.4);
}

.btn-danger {
  background: var(--danger-red);
  color: var(--white);
}

.btn-danger:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(255, 107, 107, 0.4);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-sm {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
}

.btn-block {
  width: 100%;
  justify-content: center;
}

/* Lessons Grid */
.lessons-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 2rem;
  margin-bottom: 3rem;
}

/* Lesson Card */
.lesson-card {
  background: var(--white);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: var(--transition);
  display: flex;
  flex-direction: column;
  border-left: 5px solid var(--primary-blue);
}

.lesson-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 35px rgba(77, 166, 214, 0.25);
}

.lesson-card:hover .lesson-image {
  animation: colorShift 0.6s ease-in-out;
}

@keyframes colorShift {
  0% {
    background: linear-gradient(135deg, var(--primary-blue) 0%, var(--secondary-blue) 100%);
  }
  20% {
    background: linear-gradient(135deg, #FFB6C1 0%, #FF69B4 100%);
  }
  40% {
    background: linear-gradient(135deg, #98FB98 0%, #00FA9A 100%);
  }
  60% {
    background: linear-gradient(135deg, #FFD700 0%, #FFA500 100%);
  }
  80% {
    background: linear-gradient(135deg, #87CEEB 0%, #4DA6D6 100%);
  }
  100% {
    background: linear-gradient(135deg, var(--primary-blue) 0%, var(--secondary-blue) 100%);
  }
}

.lesson-image {
  width: 100%;
  height: 200px;
  background: linear-gradient(135deg, var(--primary-blue) 0%, var(--secondary-blue) 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
  color: var(--white);
  position: relative;
  overflow: hidden;
}

.lesson-image::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.2), transparent);
}

.lesson-content {
  padding: 1.5rem;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.lesson-subject {
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--dark-blue);
  margin-bottom: 0.5rem;
}

.lesson-details {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-bottom: 1rem;
  font-size: 0.95rem;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: var(--text-light);
}

.detail-item i {
  color: var(--secondary-blue);
  width: 20px;
}

/* Shopping Cart */
.cart-container {
  display: grid;
  grid-template-columns: 1fr 350px;
  gap: 2rem;
  margin-bottom: 3rem;
}

.cart-items {
  background: var(--white);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-bottom: 1px solid var(--border-color);
  transition: var(--transition);
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.cart-item:hover {
  background: var(--light-bg);
  border-radius: 8px;
}

.cart-item-info {
  flex: 1;
}

.cart-item-name {
  font-weight: bold;
  color: var(--dark-blue);
  margin-bottom: 0.25rem;
}

.cart-item-location {
  font-size: 0.9rem;
  color: var(--text-light);
}

.cart-item-price {
  font-weight: bold;
  color: var(--secondary-blue);
  margin-right: 1rem;
}

.cart-summary {
  background: linear-gradient(135deg, var(--light-bg) 0%, #F0F9FF 100%);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border-left: 5px solid var(--secondary-blue);
  height: fit-content;
  position: sticky;
  top: 100px;
}

.summary-title {
  font-size: 1.3rem;
  font-weight: bold;
  color: var(--dark-blue);
  margin-bottom: 1rem;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border-color);
  color: var(--text-light);
}

.summary-row:last-of-type {
  border-bottom: none;
  padding-bottom: 0;
}

.summary-total {
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--secondary-blue);
  margin: 1rem 0;
}

/* Checkout Form */
.checkout-form {
  background: var(--white);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  margin: 0 auto;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: var(--dark-blue);
}

.form-group input {
  width: 100%;
}

.form-group input.error {
  border-color: var(--danger-red);
  background: #FFF5F5;
}

.error-message {
  color: var(--danger-red);
  font-size: 0.85rem;
  margin-top: 0.25rem;
  display: none;
}

.error-message.show {
  display: block;
  animation: shake 0.3s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

/* Success Message */
.success-message {
  background: #E8F5E9;
  color: var(--success-green);
  padding: 1.5rem;
  border-radius: 12px;
  border-left: 5px solid var(--success-green);
  display: none;
  animation: slideIn 0.3s ease-out;
  margin-bottom: 2rem;
}

.success-message.show {
  display: block;
}

.success-message i {
  margin-right: 0.5rem;
  font-size: 1.2rem;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 3rem 1rem;
  color: var(--text-light);
  grid-column: 1 / -1;
}

.empty-state-icon {
  font-size: 4rem;
  color: var(--border-color);
  margin-bottom: 1rem;
}

.empty-state h3 {
  color: var(--dark-blue);
  margin-bottom: 0.5rem;
}

/* Responsive */
@media (max-width: 768px) {
  .header-container {
    flex-direction: column;
    gap: 1rem;
  }

  nav {
    gap: 1rem;
    flex-wrap: wrap;
    justify-content: center;
  }

  .section-title {
    font-size: 2rem;
  }

  .lessons-grid {
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1.5rem;
  }

  .cart-container {
    grid-template-columns: 1fr;
  }

  .cart-summary {
    position: relative;
    top: auto;
  }

  .controls {
    flex-direction: column;
  }

  .control-group {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .lessons-grid {
    grid-template-columns: 1fr;
  }

  .btn {
    width: 100%;
    justify-content: center;
  }
}
</style>