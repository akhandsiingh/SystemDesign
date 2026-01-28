# Product Plan Cost Explorer System Design

## 1. Problem Statement
Design a product plan cost explorer that allows users to view pricing
plans, input usage parameters, estimate costs for a given period, and
compare plans to choose the most cost-effective option.

## 2. Assumptions
- Single product in the initial version
- Each product has multiple pricing plans
- Pricing models supported: flat fee, usage-based, tier-based
- Cost estimation is read-heavy
- No real payment or billing processing
- Users can be guest or logged-in
- Single fixed currency
- Pricing rules do not change frequently

## 3. Key Requirements

Functional Requirements
- View available pricing plans
- Input usage parameters (API calls, storage, users, etc.)
- Calculate estimated cost
- Compare multiple plans
- View cost breakdown

Non-Functional Requirements
- Low latency
- Accurate calculations
- Scalability for many users
- Easy pricing plan updates

## 4. High-Level Architecture
User → Web UI → Cost Explorer Service → Pricing Engine  
→ Cache / Database → Cost Result

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT (optional)  
Deployment: AWS EC2  

## 6. Core Components

Product
- productId
- name
- description

Plan
- planId
- productId
- name
- billingType (FLAT / USAGE / TIERED)

PricingRule
- planId
- metric (API_CALLS, STORAGE_GB, USERS)
- rate
- tierMin
- tierMax

Cost Estimator
- Calculates total cost
- Generates cost breakdown

## 7. Database Design (Short)

Plan
- plan_id (INT)
- product_id (INT)
- name (VARCHAR)
- billing_type (ENUM)

PricingRule
- rule_id (INT)
- plan_id (INT)
- metric (VARCHAR)
- rate (DECIMAL)
- tier_min (INT)
- tier_max (INT)

Index:
- (plan_id, metric)

## 8. Cost Calculation Logic

Flat Pricing:
totalCost = fixedMonthlyFee

Usage-Based Pricing:
totalCost = usage × rate

Tier-Based Pricing:
- Calculate cost for each tier based on applicable usage
- Sum tier costs to get total

## 9. Cost Comparison Flow
- User selects multiple plans
- Same usage inputs applied to all plans
- Backend calculates cost per plan
- Results returned sorted by lowest cost

## 10. API Design
GET /products  
GET /plans/{productId}  
POST /estimate  
POST /compare  

## 11. Performance Optimization
- Cache plans and pricing rules in Redis
- Stateless cost calculation
- Preload popular plans
- Minimize database access during estimation

## 12. Failure Handling
- Invalid usage input → validation error
- Missing pricing rule → fallback or error
- Cache failure → read from database
- Database failure → disable estimation

## 13. Scalability Considerations
- Aggressive caching for read-heavy workload
- Horizontal backend scaling
- Separate pricing update service
- CDN for frontend assets

## 14. Trade-Offs
- PostgreSQL for structured pricing rules
- Redis for fast reads
- Stateless services for easy scaling
- No real billing to keep system simple

## 15. Future Enhancements
- Multi-currency support
- Discounts and promo codes
- Historical cost trends
- Export reports (PDF / CSV)
- Real billing integration
- AI-based plan recommendations
