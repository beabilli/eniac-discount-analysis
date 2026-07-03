# 🚀 Eniac Discount Strategy Analysis
> A data-driven evaluation of corporate promotional policies and profit optimization.

---

## 📌 Executive Summary

### 🏢 What is Eniac?
**Eniac** is a premium, high-end technology online reseller specializing in Apple hardware, premium devices, and tech accessories. The brand positions itself in the premium market segment, with core hardware products (such as Laptops and Desktops) heavily clustered in the **€1,000 to €4,000** price range.

### 🎯 Objective of the Study
The company's Board of Directors requested a data-driven evaluation to answer one critical question: **Are discounts beneficial for Eniac?**

Our final data-driven answer is a definitive **NO**. Eniac's current permanent sale policy acts as a structural flaw. Over a 400-day historical period, the company gave away **€1.6M in discounts**, which translates into a massive **16% value loss** of the total catalog value against **€7.9M** generated in actual revenue. This study aims to isolate the financial leaks and redesign the corporate discount strategy to safeguard profit margins.

---

## 🛠️ Step-by-Step Project Development

### 📈 Notebook 1: Rigorous Data Cleaning & Structural Preprocessing Pipeline
To guarantee absolute data integrity and reliability across all database schemas, we implemented a robust, standardized data engineering pipeline designed to eliminate system bugs and structural artifacts before any analytical computing:

* **Missing Value Management (NaN Imputation & Removal):**
  * In `products_df`, missing entries in the `desc` column were dynamically backfilled by inheriting the corresponding product name.
  * Missing values in the `type` column were assigned to a temporary `"Unknown"` fallback category.
  * We surgically removed **46 corrupted records** that entirely lacked a catalog list price.
  * In `orders_df`, we dropped **5 records** missing the final checkout amount (`total_paid`) via `.dropna()` to eliminate any distortion in downstream financial metrics.
* **Deduplication:** We scanned all dataframes (`orders`, `orderlines`, `products`) and performed a thorough deduplication process to eliminate identical records and avoid double-counting revenues.
* **Text Normalization & Standard Trimming:** Applied `.str.strip()` across all textual and categorical columns in every single DataFrame to eliminate invisible trailing or leading whitespaces that would otherwise cause faulty string joins or erroneous aggregations.
* **Datetime Standardization:** Explicitly parsed and converted all string-based chronological columns into native Pandas datetime objects (`DatetimeIndex`) via `pd.to_datetime()`, enabling high-performance time-series operations.
* **Corrupted Price Repair via Regular Expressions (Regex):**
  * In both `orderlines_df` and `products_df`, we intercepted and repaired a critical database formatting bug characterized by a *"three-dot anomaly"* (e.g., formats like `nn.nnn.nnn`).
  * Using precise Regex string manipulation, we purged the fictitious separators and re-established the correct floating-point decimal scale necessary for mathematical computations.
  * Enforced a strict `unit_price > 0` relational mask to discard zero-value or negative-value transactional anomalies.
* **Feature Pruning:** Streamlined memory allocation and dismissed redundant dimensions by dropping the `product_id` column in `orderlines_df` (which consisted entirely of zeros) and the `promo_price` column in `products_df` due to severe data incompleteness.

### 📈 Notebook 2: Data Synchronization & Financial Outlier Removal

**Dataset Time Horizon:** Before starting the cross-table synchronization and the shopping cart validation, we calculated the total time horizon of the data. By computing the difference between the maximum and minimum transaction dates, we verified that the dataset covers a historical period of **437 days**, providing a solid chronological baseline for our study.

#### 🔄 Step 1: Referential Integrity & Table Synchronization
In this stage, we prepared the relational data links, isolated the real financial streams, and applied statistical modeling to answer the Board's specific inquiries regarding checkout variances and product discount distributions:

* **Isolation of the Real Financial Flow:** Applied a strict relational mask to retain exclusively successful transactions (`state == 'Completed'`), surgically excluding abandoned carts, failed transaction attempts, and canceled orders from any revenue computation.
* **Standardization of Merge Keys:** We renamed `id_order` to `order_id` in `orderlines_df` to match the `orders` schema.
* **Bidirectional Synchronization:** We synchronized `orders_df` and `orderlines_df` using the `.isin()` method. This step ensured that we only retained orders that existed simultaneously in both dataframes, eliminating orphaned rows or ghost transactions.
* **Corrupted Orders Elimination (SKU Alignment):** We identified products sold in `orderlines` whose SKU was missing from the master product catalog (`products_df`). To protect data consistency, we extracted the unique `order_ids` of these corrupted transactions and completely wiped them from both dataframes, preventing misaligned total values and skewed revenue calculations.

#### 🛒 Step 2: Shopping Cart Validation & Outlier Removal (IQR on Price Discrepancies)
* **Cart vs. Paid Analysis:** We calculated the total theoretical value of each cart (`unit_price * qty`) and merged it back into the main orders dataframe. By subtracting the cart total from the actual amount paid (`total_paid - unit_price_total`), we isolated the net price differences (`price_diff`) representing shipping fees (positive values) or cart-wide coupons (negative values).
* **Advanced Outlier Filtering:** To eliminate extreme anomalies, system logging bugs, or massive data corruptions, we calculated the Interquartile Range (IQR) specifically on these price differences.
* **Data Preservation:** By removing records outside the lower and upper bounds (Q1 - 1.5 × IQR and Q3 + 1.5 × IQR), we retained a highly consistent, statistically sound dataset, ensuring our final revenue metrics were completely unskewed.

#### 📊 Board Inquiries & Answers Post-Statistical Cleanup:
* **Mean Discrepancy:** The actual mean price difference stabilizes at **€3.94**.
* **Variance Distribution:** The cleaned data robustly clusters around three standardized commercial tiers:
  * **25th Percentile:** €0.00 (representing promotional or free shipping thresholds).
  * **50th Percentile (Median):** Exactly €4.99 (Eniac's baseline shipping tier).
  * **75th Percentile:** Exactly €6.99 (Eniac's premium shipping tier).
  * The absolute minimum and maximum clean values are established at **-€5.00** and **€16.97** respectively.
* **Are all differences explained by shipping costs?** **Yes.** Post-cleanup, every data point aligns with e-commerce logic. The fixed peaks at €4.99 and €6.99 perfectly match corporate shipping matrixes. The €16.97 maximum is entirely compatible with express, heavy, or insured deliveries. Negative values down to -€5.00 capture the application of cart-level discount coupons at checkout.
* **Handling Unexplainable Outliers:** Severe anomalies (such as the +€3,984 bug) were mathematically pruned via IQR to protect the integrity of corporate financial metrics.
* **Total Real Cash Revenue:** By aggregating the `total_paid` attribute for all valid completed orders within `comparison_df`, Eniac's absolute gross cash intake for the period is verified at **7,978,675.60 €**.

#### 🏷️ Step 3: Product-Level Discount Analysis (Unit Price vs. List Price)
We evaluated individual item markdown strategies by joining `orderlines_df` (`unit_price`) with `products_df` (`price`). In this new merged table (`orderlines_products`), we generated the absolute discount metric:

`orderlines_products["discount_euro"] = orderlines_products["price"] - orderlines_products["unit_price"]`

Initial distributions of this column were heavily corrupted by original database typographical errors, displaying impossible negative discounts down to -€560 and absurd positive anomalies exceeding €8,000.
To resolve this and prevent severe mathematical distortions caused by catalog products with a zero-list-price artifact, we applied the IQR (1.5x) filter directly on markdown percentages to clean the catalog, yielding the following strategic results:

* **True Average Discount:** The actual average real discount per product stabilizes at a commercially sound **€18.48**.
* **The Median Markdown:** The median absolute discount sits at **€14.00**, confirming that Eniac's operational baseline relies heavily on continuous promotional campaigns.
* **The Negative Discount Phenomenon (-33.64%):** The absolute minimum percentage value of -33.64% reveals a highly interesting business dynamic. This represents a specific niche segment of products sold at a premium (surcharge/markup), directly tied to severe product scarcity or direct retailer-end hardware customizations (such as RAM or SSD upgrades pre-installed by the reseller before shipping).

#### 📊 Discount Volume and Operational Distribution

* **Volume of Discounted Products:** Our calculation reveals a staggering metric: **95.94%** of the total products sold by Eniac carry a price markdown. Out of the entire cleaned transaction database, only a minimal 4.06% of items were sold at full retail price. This structural dependency mathematically proves that Eniac does not rely on standard retail value conversions; instead, the company's business model is completely dependent on permanent sales campaigns to trigger customer transactions.
* **Advanced IQR Application on Percentages:** To ensure that our visual analysis was completely accurate, we applied a secondary Interquartile Range (IQR 1.5x) filter directly on the discount percentage column. This surgical cleanup successfully pruned unrealistic percentage records and extreme markdown outliers, protecting the final frequency calculation from statistical distortions.

<img width="870" height="455" alt="image" src="https://github.com/user-attachments/assets/6a79e2f0-cb6f-4a7e-8c68-e6119b00cdc5" />

* **Discount Size and Range:** Post-cleanup, the discount percentages predominantly cluster within a commercially sound range between **0% and 25%**, showing that Eniac’s standard promotional policy focuses on moderate price cuts to attract buyers without completely depleting product margins.
* **Statistical Distribution Shape:** The histogram shows a heavily right-skewed (positively skewed) distribution. Frequency rises immediately at 0%, maintains a high plateau up to 25%, and then continuously drops. Markdowns exceeding 30% are limited, and extreme discounts above 50% are rare exceptions.
* **Negative Distribution Tail:** The visualization displays a small subset of negative percentages down to around -20%. This directly aligns with our prior findings on custom premium surcharges and scarcity-driven markups, showing that Eniac successfully charges a premium on specific hardware modifications.

---

#### 📅 Step 4: Seasonal Trends Analysis
* **Methodology:** We aggregated our validated checkout data on a monthly chronological scale. By cross-referencing the monthly sum of total paid transactions against the absolute count of completed orders, we mapped the operational volume against the generated cash intake.
* **The March Data Drop:** Our timeline identifies an extreme drop in March 2017, where completed orders plummeted to just 163, dragging the revenue line close to the zero axis. This anomalous contraction represents a technical database logging bug or missing source records rather than an actual commercial shutdown.
* **Natural Seasonal Synchronization:** Post-cleanup, the visualization demonstrates an almost perfect synchronization between order volumes and financial intake. The absolute historical peak is achieved in November 2017, where both revenue and order counts reach maximum capacity due to Black Friday demand.

<img width="1184" height="484" alt="image" src="https://github.com/user-attachments/assets/1512858b-0e94-4cb1-892e-024fce04116f" />

---

#### 📊 Step 5: Promotions Impact Analysis
* **Methodology:** We integrated our monthly financial performance with corporate markdown data, plotting Monthly Revenue as clear vertical bars against the Average Discount Rate as a continuous line.
* **The Strategic Breakdown:** The analysis reveals a complete lack of correlation between aggressive discount increments and revenue acceleration outside of peak periods. This is heavily illustrated by the **"July Trap"**, where Eniac offered its highest discount rate of the entire period (**25%**), but revenue stagnated deeply under **0.5M €**.
* **Seasonality over Price Cuts:** The chart mathematically proves that natural seasonal demand drives corporate growth, not permanent price-slashing. In **November**, a 22% discount successfully synchronized with Black Friday demand to reach a historical record of **1.2M €**. Conversely, in **December**, a lower discount of **19%** still yielded massive Christmas sales.
* **Business Insight:** Discounts act as accelerators for existing seasonal demand, but they completely fail to stimulate artificial volume during low-season months, leading only to margin destruction.

<img width="1084" height="483" alt="image" src="https://github.com/user-attachments/assets/a123772a-254c-4086-9577-d198ee813b25" />

* **Strategic Conclusion:** This mathematical correlation confirms that the Black Friday spike represents a highly profitable cash injection. The deep November discounts did not cannibalize revenue; instead, they triggered an extraordinary volume expansion that successfully converted high shipping traffic into massive liquid value.
* **The July Trap:** In stark contrast, July experienced the absolute maximum discount rate of the year (over 25%), yet revenue cratered below €500,000. During cold shopping cycles, excessive discounting fails to stimulate consumer behavior; it fails to scale volume and only serves to destroy operational margins while devaluing inventory.

---

## 📊 Notebook 3: Category Performance & Discount Cap Optimization

#### 🏷️ Text-Based Product Categorization & Hierarchy
To find exactly where Eniac is losing its 16% revenue, we broke down the raw catalog into 14 distinct product categories. Since the database was messy, we developed an automated text-matching model. We implemented a Python pipeline using the Pandas library, structured into three logical phases:

##### 1. Keyword Extraction (Using the `+=` Operator)
* We initialized a new empty string column named `"category"` inside our DataFrame.
* We then applied the `.str.contains()` method to scan all product names and descriptions.
* During this step, using the `+=` operator was crucial. This technical choice allowed us to "append" multiple tags to the same item if it contained multiple keywords, helping us flag hybrid products.
* To ensure code stability and prevent system crashes caused by missing values, we integrated the safety parameter `na=False`.

##### 2. Priority Management and Conflict Resolution
* Appending tags generated several comma-separated combinations, such as `", Laptop, Storage"` for a MacBook Pro mentioning its internal drive.
* To untangle these overlaps and clean the database, we applied a strict Corporate Hierarchy using `.loc` statements and the standard `=` operator:
  * **The Core Hardware Rule:** To prevent main hardware from being downgraded to components, primary devices like Laptops, Smartphones, and Tablets always override accessory tags. For example, a MacBook Pro text mentioning "1TB SSD" must strictly remain a Laptop and not become Storage.
  * **The Specific Accessories Shield:** Using boolean logical masks, we isolated cases where hardware was mentioned only as compatibility (such as a "Backpack for MacBook" or an "SSD Upgrade Kit for MacBook"). In these specific scenarios, accessory keywords like *case*, *kit*, or *upgrade* bypassed the general rule, forcing the item into Protection or Storage. This prevented €30 items from skewing the average price of €2,000 computers.
  * **The Services Regulation:** If a service conflicted with hardware (such as an iPhone repair), the Service category won, since from a business perspective it represents technical labor rather than a physical good sale.
  * **Disambiguating Similar Strings:** We resolved textual overlaps between categories with identical prefixes, like Smartwatch and Smart_home, isolating their specific patterns to avoid misalignments during overwrites.

##### 3. String Optimization and the Catch-All Category
* Once all conflicts were resolved, we used the native Pandas `.str.lstrip(", ")` function to surgically remove the leading comma and space from clean cells, turning values like `", Laptop"` into `"Laptop"`.
* Finally, any products that were completely left out of our keyword filters were assigned to the catch-all category named **Other**.

---

##### 📦 Final Standardized Commercial Categories:
The text-matching model successfully reorganized the entire messy database into **14 standardized business departments**:

* Audio & Video
* Cable & Adapter
* Desktop
* Ipod
* Keyboard
* Laptop
* Other (Catch-all)
* Protection
* Service
* Smart home
* Smartphone
* Smartwatch
* Storage
* Tablet

--- 

#### 📊 Plot 1: Financial Ranking and Price Distribution Analysis
* **Cleaned Revenue Ranking:** We aggregated the total revenue across all categories to identify the primary financial engines of the company, verifying that the final consolidated clean revenue successfully converges at the corporate baseline.
* **Price Distribution via Boxplots:** To evaluate the pricing structure of each department, we plotted a Boxplot showing the distribution of original list prices across all 14 categories.
* **Strategic Pricing Insights:** The Boxplot mathematically confirms Eniac’s high-end tech identity. Core hardware categories, specifically Laptops and Desktops, show data points heavily clustered at very high price positions (**€1,000 to €3,000**).
* **The Customer Profile:** This visualization proves that clients buying from these premium segments are high-budget shoppers driven by necessity and brand loyalty, meaning they do not require aggressive discounts to finalize their purchase.

<img width="1389" height="690" alt="image" src="https://github.com/user-attachments/assets/6c276ed4-786f-4482-a642-0e35441a20e2" />

---

#### 📉 Plot 2: Strategic Category Breakdown (Performance & Markdown Alignment)
* **Methodology:** We generated a comprehensive three-dimensional visualization combining operational volume as light blue bars, absolute revenue as dark blue label tags, and average promotional strategies as a continuous orange line.
* **The High-Value Powerhouses:** Our primary income engines are **Storage (€3.03M)** and **Smartphones (€1.42M)**. These core categories drive over **55% of Eniac’s total wealth** while maintaining our lowest historical average discount rates, hovering between **14% and 16%**.
* **The Traffic Hooks:** Low-cost departments like *Protection* and *Cable & Adapter* carry heavy promotional markdowns exceeding **30%**. This aggressive discounting drives massive unit sales volume but generates very low revenue, proving they act successfully as online traffic drivers to enhance customer acquisition.
* **The Structural Failure:** The analysis highlights a critical strategic leak in the *Laptop* and *Desktop* departments. Despite receiving a permanent, aggressive **21% average discount** that destroys profit margins, computer sales volumes remain completely flat and generate a poor **€345K** in revenue. Price-slashing fails to stimulate hardware demand.

<img width="1183" height="584" alt="image" src="https://github.com/user-attachments/assets/9e20d3ef-71f3-4da3-b60f-af11b9849b3a" />

---

#### 📊 Plot 3: Baseline Revenue and Sales Volume Comparison
* **Methodology:** We structured a dual-axis visualization to study the pure baseline distribution of Eniac’s commercial departments, plotting Total Revenue Generated as clean vertical bars against Total Units Sold as a synchronized continuous line.
* **The Core Revenue Engines:** The chart visually highlights the absolute dominance of **Storage (€3.03M)** and **Smartphone (€1.42M)**. These two departments stand as the primary core categories driving the vast majority of Eniac's total financial wealth.
* **The Volume Paradox:** The visualization introduces a major operational paradox. Categories like *Protection* show an aggressive peak in units sold (reaching nearly 10,000 items), yet their corresponding revenue bar remains extremely low, indicating high operational effort for low financial return.
* **The High-Ticket Item Gap:** Conversely, *Laptops* and *Desktops* show a flat trend; despite being premium hardware segments with high list prices, their total sales volume and overall revenue contribution remain heavily compressed at the bottom of the chart.

<img width="1184" height="584" alt="image" src="https://github.com/user-attachments/assets/e9d766f4-7aa8-41f3-b30f-0028cf9faa71" />

---

## 🏆 Strategic Action Plan & Commercial Recommendations
* **The Core Strategy:** Based on our empirical data and financial audit, we designed a comprehensive action plan to stop the hemorrhage of **€1.6 million** in corporate losses. Eniac must move away from generic, continuous site-wide markdowns and shift toward a highly differentiated promotional model based on strict operational caps and clear calendar boundaries.

### 📅 1. Recommendations About Seasonality
* **Off-Peak Months Management:** Eniac must maintain minimal maintenance discounts (**below 10-12%**) during off-peak months. This rule applies directly to **July** (when consumers are on vacation), **January** (post-holiday cooldown), and **March** (historically low purchase intent). Outside of peak windows, aggressive price-slashing completely fails to create artificial consumer demand; it only burns margins.
* **Peak Seasonal Windows Execution:** Corporate price cuts **higher than 15%** must be reserved exclusively for peak seasonal windows, such as **Black Friday** and **Christmas**. During these high-intent quarters, immense sales volumes successfully cover the price reduction and drive peak profitability.

### 📊 2. Recommendations About Categories and Discount Caps
* **💻 Premium Hardware (Laptops & Desktops) ➔ MAX CAP: 0% (Full List Price):** Discounting laptops at 21% generates a mere €345,000 and does not increase sales volumes at all. Customers looking for high-end Apple hardware have both the budget and the immediate need for it. Discounting these high-ticket items simply gives away €200 to €500 in profit per customer. Eniac must stop all hardware sales and sell at full price.
* **📱 Revenue Drivers (Storage & Smartphones) ➔ MAX CAP: 15%:** These categories alone generate **55% of Eniac's total wealth (€4.4M)** and sell consistently with very low discount rates (14-16%). There is absolutely no business need to exceed a 15% average discount because organic demand is already extremely high; protecting this margin is critical.
* **🔌 Loss Leaders / Traffic Builders (Protection & Cables) ➔ NO CAP (Up to 30-35%):** Heavily discounting covers and cables (~30%) drives massive inventory volume, even though it brings in very little actual revenue. Discounting a cable by 30% costs the company only €3 but attracts thousands of users to the website. This aggressive strategy should be actively maintained to boost SEO ranking and drive massive web traffic.

---

## 🎯 Executive Conclusions: Strategy Over Habit
* **The Core Truth:** Across seasons, categories, and our best-performing product line, deeper discounts did not reliably produce more revenue.
* **The Cultural Issue:** For Eniac, discounting has become a habit, not a strategy. Price-slashing has been used as a permanent operational crutch rather than a calculated tactical choice.
* **The Final Verdict:** The data does not support that it's paying for itself. Burning €1.6 million in margins has failed to accelerate corporate growth. Eniac must immediately reclaim its premium identity, enforce strict discount caps, and shift toward dynamic, seasonal pricing to restore corporate profitability.

---

## 💻 Technologies Used
* **Python 3**
* **Pandas** (Data Manipulation)
* **Matplotlib & Seaborn** (Data Visualization)
* **Jupyter Notebook**
