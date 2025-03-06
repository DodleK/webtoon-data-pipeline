# **Webtoon Data Engineering Project Documentation**  

## **1. Project Overview**  
### **Objective**  
This project is designed to **scrape, clean, and store** Webtoon manga data from a website. It extracts key details such as **title, genre, rating, views, subscribers, release date, and synopsis**. The processed data is then saved in a structured format for further analysis.  

### **Key Features**  
- **Web Scraping:** Extracts Webtoon manga data using `requests` and `BeautifulSoup`.  
- **Data Cleaning & Processing:** Ensures consistency using `pandas` and `numpy`.  
- **Data Storage:** Saves structured data in **MySQL Server**.  
- **ETL Pipeline:** Implemented **Apache Airflow** for automation.  
- **Visualization:** Data will be analyzed and visualized using **Power BI**.  

## **2. Tools & Technologies Used**  
| **Category**       | **Tool/Technology** |
|--------------------|--------------------|
| **Scraping**      | `requests`, `BeautifulSoup` |
| **Data Processing** | `pandas`, `numpy` |
| **Database**       | MySQL Server |
| **ETL Pipeline**   | Apache Airflow |
| **Visualization**  | Power BI |

## **3. Data Source**  
- **Website:** [Webtoon Originals](https://www.webtoons.com/en/originals)  
- **Scraped Data Fields:**  
  - `title_id`: Unique ID of the Webtoon  
  - `title`: Name of the Webtoon  
  - `released_date`: First episode release date  
  - `genre`: Webtoon genre  
  - `authors`: Creator(s) of the Webtoon  
  - `weekdays`: Release schedule (e.g., Monday, Friday)  
  - `length`: Total number of chapters  
  - `subscriber`: Number of subscribers  
  - `rating`: Average user rating  
  - `views`: Total views count  
  - `likes`: Number of likes  
  - `status`: Whether the Webtoon is **Ongoing, Completed, or on Hiatus**  
  - `daily_pass`: Availability of daily free episodes  
  - `synopsis`: Short summary of the Webtoon  

## **4. Project Workflow**  
### **Step 1: Web Scraping**  
- **Fetch HTML Content:**  
  - Used `requests.get()` to send an HTTP request to the Webtoon page.  
  - Included **headers** (User-Agent, Accept-Language) to avoid blocking.  

- **Parse HTML with BeautifulSoup:**  
  - Extracted **Webtoon links** from the main page using `soup.find_all()`.  
  - Iterated through each link and extracted **manga details**.  

### **Step 2: Data Extraction**  
For each Webtoon page, the following functions are used to extract information:  

| **Function Name** | **Purpose** |
|------------------|------------|
| `get_title_id(link)` | Extracts the unique Webtoon ID from the URL. |
| `get_title(soup)` | Extracts the Webtoon title. |
| `get_released_date(soup)` | Retrieves the first episode's release date. |
| `get_genre(soup)` | Extracts the genre of the Webtoon. |
| `get_authors(soup)` | Retrieves the list of authors/creators. |
| `get_weekdays(soup)` | Extracts the release schedule. |
| `get_length(soup)` | Determines the number of chapters available. |
| `get_subscriber_count(soup)` | Retrieves the total number of subscribers. |
| `get_rating(soup)` | Extracts the user rating of the Webtoon. |
| `get_views_count(soup)` | Retrieves the total number of views. |
| `get_likes_count(main_soup, link)` | Extracts the number of likes. |
| `get_status(main_soup, link)` | Determines if the Webtoon is **Ongoing, Completed, or on Hiatus**. |
| `get_daily_pass(soup)` | Checks if the Webtoon has daily free episodes. |
| `get_synopsis(soup)` | Extracts a short summary of the Webtoon. |


### **Step 3: Data Cleaning & Transformation**  
- Used `pandas` to store the extracted data in a **dictionary** and then converted it into a **DataFrame**.
- Dropped duplicate **`title_id`** values to ensure there are no duplicate entries in the dataset.
- Converted **numeric values** (ratings, views, likes, subscribers) to appropriate data types.  
- Removed special characters and unnecessary whitespace from **text fields** for better readability..
- Converted the **`released_date`** column to the **date datatype** to maintain consistency and ensure proper analysis.

Here's the updated **Data Storage** section with your table schema included:


### **Step 4: Data Storage**  
- The cleaned data is stored in a **Microsoft SQL Server**.
- **Table Schema:**  
  - A **WebtoonData** table was created in SQL Server with the following schema:
  
    ```sql
    CREATE TABLE WebtoonData (
        title_id INT PRIMARY KEY,                -- Unique identifier (Cannot be NULL)
        released_date DATE NOT NULL,             -- Date of release (Cannot be NULL)
        title NVARCHAR(255) NOT NULL,            -- Webtoon title (Cannot be NULL)
        genre NVARCHAR(100) NOT NULL,            -- Genre (Cannot be NULL)
        authors NVARCHAR(255) NOT NULL,          -- Authors (Cannot be NULL)
        weekdays NVARCHAR(50) NOT NULL,          -- Publishing days (Cannot be NULL)
        length INT CHECK (length > 0),           -- Must be positive
        subscriber INT NOT NULL DEFAULT 0,       -- Default 0 if no subscribers
        rating FLOAT CHECK (rating BETWEEN 0 AND 10), -- Ensure rating is between 0-10
        views BIGINT NOT NULL DEFAULT 0,         -- Ensure views default to 0
        likes BIGINT NOT NULL DEFAULT 0,         -- Ensure likes default to 0
        status NVARCHAR(50) NOT NULL,            -- Ongoing, Completed, etc.
        daily_pass NVARCHAR(50) NOT NULL,         -- True, False
        synopsis NVARCHAR(MAX) NOT NULL          -- Webtoon description (Cannot be NULL)
    );
    ```

- **SQLAlchemy Connection:**  
  - Established a connection to **Microsoft SQL Server** using **SQLAlchemy**.
  - The connection string follows this format:
  
    ```python
    engine = sql.create_engine('mssql://<username>:<password>@<server>/<database>?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
    ```

- **Loading Data into SQL Server:**  
  - Used the `to_sql()` function to **append** the cleaned data into the **WebtoonData** table in SQL Server.
  - The **`if_exists='append'`** parameter ensures that new data is added to the existing table without overwriting it.
  - The `index=False` ensures that the DataFrame's index is not saved as a separate column in the SQL table.
  
    ```python
    df_unique.to_sql('WebtoonData', con=conn, index=False, if_exists='append')
    ```


### **Step 5: Automation & Visualization**  (Future Work) 
- **Apache Airflow** is used to schedule daily/weekly data extraction.  
- **Power BI** is used to visualize **trending Webtoons, most subscribed genres, and popular authors**.  

## **5. Challenges & Solutions**  
| **Challenge** | **Solution** |
|--------------|-------------|
| Website blocking requests | Used **User-Agent headers** to mimic a real browser. |
| Dynamic content loading | Consider using **Selenium** for JavaScript-heavy pages. |
| Inconsistent data formats | Standardized missing values and numeric conversions. |
| Large data storage | Migrated to **MySQL Server** for better handling. |

## **6. Future Improvements**  
- **Expand Scraping Scope:** Include **multiple Webtoon platforms**.  
- **Improve Data Storage:** Optimize indexing and partitioning in MySQL for large datasets.  
- **Build a Web API:** Create an **API endpoint** to fetch Webtoon data dynamically.  
- **Deploy a Dashboard:** Use **Power BI** to visualize insights on popular Webtoons.  

## **7. Conclusion**  
This project successfully **scrapes, processes, and stores** Webtoon data in **MySQL Server** for further analysis. Future enhancements will improve automation, data storage, and visualization.    

