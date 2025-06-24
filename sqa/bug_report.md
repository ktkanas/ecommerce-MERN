# Bug Report - FashionHub E-commerce Application

**Report Date:** September 2, 2024 
**Reported by:** Muhammad Anas
**Test Environment:** Staging (https://staging.fashionhub.com)  
**Build Version:** v2.3.1-staging  
**Browser:** Chrome 125.0.6422.141, Firefox 126.0, Safari 17.4  

---

## BUG SUMMARY DASHBOARD

| Severity | Count | Status |
|----------|-------|--------|
| Critical | 2 | 🔴 Open |
| High | 5 | 🔴 Open |
| Medium | 7 | 🟡 Open |
| **Total** | **14** | **Under Investigation** |

---

## CRITICAL SEVERITY BUGS

### 🔴 BUG-006: Payment Processing Timeout
**Priority:** P1 - Critical  
**Severity:** Critical  
**Status:** Open  
**Assigned to:** Backend Team Lead  
**Reporter:** Muhammad Anas   
**Date Found:** September 2, 2024 

**Environment:** Staging  
**Browser:** All browsers  
**Device:** Desktop and Mobile  

#### Summary
Payment processing consistently times out after 30 seconds when using credit card payments, preventing customers from completing purchases.

#### Steps to Reproduce
1. Add products to cart (any products)
2. Proceed to checkout
3. Fill shipping information completely
4. Select "Credit Card" as payment method
5. Enter test card details:
   - Card Number: 4242424242424242
   - Expiry: 12/26
   - CVV: 123
   - Cardholder Name: John Doe
6. Click "Pay Now" button
7. Wait for response

#### Expected Result
Payment should process within 10 seconds and show success confirmation

#### Actual Result
- Payment processing indicator shows for 30 seconds
- Timeout error appears: "Payment processing failed. Please try again."
- No error logging visible in browser console
- User remains on payment page with form data intact

#### Impact Assessment
- **Business Impact:** CRITICAL - Complete revenue loss from online sales
- **User Experience:** Severe - Customers cannot complete purchases
- **Frequency:** 100% reproduction rate
- **Affected Users:** All customers attempting to pay

#### Screenshots
![Payment Timeout Error](screenshot-payment-timeout.png)  
![Payment Processing Spinner](screenshot-payment-loading.png)

#### Additional Information
- Issue occurs with all test credit cards
- Same issue observed with real card numbers (tested with personal card)
- PayPal payment option works correctly
- Network tab shows request to `/api/payment/process` hangs indefinitely
- Server logs show connection timeout to Stripe API

#### Possible Root Cause
- Stripe API integration configuration issue
- Firewall blocking outbound HTTPS requests to Stripe
- Invalid API keys in staging environment
- Missing environment variables for payment processing

#### Recommended Fix Priority
**IMMEDIATE** - This is blocking all credit card transactions

---

### 🔴 BUG-012: Server Performance Degradation Under Load
**Priority:** P1 - Critical  
**Severity:** Critical  
**Status:** Open  
**Assigned to:** DevOps Team  
**Reporter:** Ahsan Qadir  
**Date Found:** June 24, 2025  

**Environment:** Staging  
**Load Testing Tool:** Artillery.io  

#### Summary
Application becomes completely unresponsive when more than 75 concurrent users access the system, causing 500 server errors and connection timeouts.

#### Steps to Reproduce
1. Set up load testing with Artillery
2. Configure test for 100 virtual users
3. Each user performs: Browse products → Search → Add to cart → Checkout
4. Run test for 5 minutes
5. Monitor server response times and error rates

#### Expected Result
System should handle 1000 concurrent users as per performance requirements with <2% error rate

#### Actual Result
- At 50 users: Response time increases to 5-8 seconds
- At 75 users: Server starts returning 500 errors
- At 80+ users: Complete system unresponsiveness
- Database connection pool exhausted
- Memory usage spikes to 98%

#### Performance Metrics
| Concurrent Users | Avg Response Time | Error Rate | Memory Usage |
|------------------|-------------------|------------|--------------|
| 10 | 1.2s | 0% | 45% |
| 25 | 2.1s | 0.5% | 62% |
| 50 | 5.8s | 3% | 78% |
| 75 | 12.3s | 15% | 94% |
| 100 | Timeout | 85% | 98% |

#### Impact Assessment
- **Business Impact:** CRITICAL - Site unusable during peak traffic
- **Revenue Impact:** High - Lost sales during high-traffic periods
- **User Experience:** Severe - Site crashes under normal load
- **Scalability:** Failing to meet basic performance requirements

#### Server Logs Extract
```
[ERROR] 2025-06-24 14:23:15 - Database connection timeout
[ERROR] 2025-06-24 14:23:16 - MongoDB connection pool exhausted
[ERROR] 2025-06-24 14:23:17 - Memory limit exceeded: 2048MB
[FATAL] 2025-06-24 14:23:18 - Application crash: Out of memory
```

#### Root Cause Analysis
- **Database Connections:** MongoDB connection pool limited to 10 connections
- **Memory Leaks:** React components not properly unmounting
- **Inefficient Queries:** N+1 query problem in product listing
- **No Caching:** Database queries not cached, hitting DB for every request
- **Resource Limits:** Server allocated only 2GB RAM, insufficient for load

#### Recommended Solutions
1. **Immediate (24 hours):**
   - Increase MongoDB connection pool to 100 connections
   - Increase server memory to 8GB
   - Implement Redis caching for product queries

2. **Short-term (1 week):**
   - Fix memory leaks in React components
   - Optimize database queries with proper indexing
   - Implement connection pooling

3. **Long-term (1 month):**
   - Implement horizontal scaling with load balancer
   - Move to microservices architecture
   - Implement CDN for static assets

---

## HIGH SEVERITY BUGS

### 🟠 BUG-002: Product Detail Page Not Loading Properly
**Priority:** P2 - High  
**Severity:** High  
**Status:** Open  
**Assigned to:** Frontend Team  
**Reporter:** Ahsan Qadir  
**Date Found:** June 24, 2025  

#### Summary
Product detail pages fail to load product images and size selectors, preventing customers from viewing complete product information.

#### Steps to Reproduce
1. Navigate to products page (https://staging.fashionhub.com/products)
2. Click on any product to open detail page
3. Observe product image section
4. Check size/color selector dropdowns

#### Expected Result
- Product images display in gallery format
- Size selector shows available sizes
- Color options visible
- Add to cart button functional with selections

#### Actual Result
- Product images show broken image placeholders
- Size selector dropdown is empty
- Color selector missing entirely
- Add to cart button disabled

#### Impact Assessment
- **Conversion Impact:** High - Customers cannot make informed purchases
- **User Experience:** Poor - Essential product information missing
- **SEO Impact:** Medium - Product pages not properly indexed

#### Browser Console Errors
```javascript
GET https://staging.fashionhub.com/api/products/images/IMG_001.jpg 404 (Not Found)
TypeError: Cannot read property 'sizes' of undefined at ProductDetail.jsx:45
Warning: Failed prop type: The prop 'colors' is marked as required in ColorSelector
```

#### Root Cause
- Image URLs pointing to incorrect CDN path
- Product data model missing size/color arrays
- React component expecting different data structure

---

### 🟠 BUG-004: Cart Price Calculation Incorrect
**Priority:** P1 - High  
**Severity:** High  
**Status:** Open  
**Assigned to:** Backend Team  
**Reporter:** Ahsan Qadir  
**Date Found:** June 24, 2025  

#### Summary
Shopping cart displays incorrect total prices when item quantities are updated, leading to pricing discrepancies.

#### Steps to Reproduce
1. Add product to cart (price: $29.99)
2. Navigate to cart page
3. Change quantity from 1 to 3
4. Click "Update" button
5. Observe total price calculation

#### Expected Result
Total should be: $29.99 × 3 = $89.97 (plus applicable taxes)

#### Actual Result
- Quantity updates to 3 correctly
- Total price remains $29.99
- Tax calculation based on incorrect subtotal
- Checkout page shows same incorrect total

#### Impact Assessment
- **Financial Impact:** Critical - Potential revenue loss
- **Legal Impact:** High - Incorrect pricing could cause disputes
- **Customer Trust:** High - Pricing errors damage credibility

#### Code Analysis
Located in `/client/src/components/Cart/CartItem.jsx` line 78:
```javascript
// BUG: Total not recalculated when quantity changes
const calculateTotal = () => {
  return price; // Should be: price * quantity
}
```

---

### 🟠 BUG-008: Order History Not Displaying
**Priority:** P2 - High  
**Severity:** High  
**Status:** Open  
**Assigned to:** Backend Team  
**Reporter:** Muhammad Anas 
**Date Found:** September 2, 2024 

#### Summary
Users cannot view their order history despite having completed orders, affecting customer service and user experience.

#### Steps to Reproduce
1. Login as user: testuser@fashionhub.com
2. Navigate to "My Account" → "Order History"
3. Observe page content

#### Expected Result
Display list of user's previous orders with:
- Order number
- Order date
- Items purchased
- Order total
- Order status

#### Actual Result
Page displays "No orders found" message even though user has 3 completed orders in database

#### Database Verification
Manual database query confirms orders exist:
```sql
db.orders.find({"userId": "60f7b3c4d5e8f9a1b2c3d4e5"})
// Returns 3 orders for test user
```

#### API Response Analysis
GET `/api/users/orders` returns:
```json
{
  "success": true,
  "orders": [],
  "message": "No orders found"
}
```

#### Root Cause
Backend API query filtering by incorrect user ID field - using `user_id` instead of `userId`

---

### 🟠 BUG-013: JWT Token Security Vulnerability
**Priority:** P2 - High  
**Severity:** High  
**Status:** Open  
**Assigned to:** Security Team  
**Reporter:** Ahsan Qadir  
**Date Found:** June 24, 2025  

#### Summary
JWT tokens never expire and remain valid even after user logout, creating security vulnerabilities.

#### Steps to Reproduce
1. Login and capture JWT token from localStorage
2. Logout from application
3. Use captured token to make API requests
4. Check token validity after 24+ hours

#### Expected Result
- Tokens should expire after 24 hours
- Tokens should be invalidated on logout
- Expired tokens should be rejected by API

#### Actual Result
- Tokens never expire (no expiration time set)
- Tokens remain valid after logout
- Old tokens continue to work indefinitely

#### Security Impact
- **High Risk:** Compromised tokens can be used indefinitely
- **Session Hijacking:** Increased risk of unauthorized access
- **Compliance Issue:** Violates security
