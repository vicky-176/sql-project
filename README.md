# sql-project
-- Basic Joins

-- 1. Retrieve all movies along with their directors.
select names.name,movie.title from movie inner join director_mapping on movie.id=director_mapping.movie_id inner join names on names.id=director_mapping.name_id;

-- 2. Get all movies and their genres.
select genre.genre,movie.title from movie inner join genre on movie.id=genre.movie_id;

-- 3. List all actors along with the movies they have acted in.
select names.name,movie.title from movie inner join role_mapping on movie.id=role_mapping.movie_id inner join names on names.id=role_mapping.name_id where role_mapping.category="actor";

-- 4. Get all movies and their corresponding writers.
select title,production_company from movie;

-- 5. Retrieve all users who have rated movies along with the rating they provided.
select ratings.total_votes,ratings.avg_rating,movie.title from movie inner join ratings on movie.id=ratings.movie_id;


-- Inner Joins

-- 1. Find the names of actors who have acted in at least one movie.
select names.name,count(movie.title) from names inner join role_mapping on names.id=role_mapping.name_id inner join movie on movie.id=role_mapping.movie_id where role_mapping.category="actor" group by names.name having count(movie.title)>=1;

-- 2. List all directors who have directed more than 5 movies.
select names.name,count(movie.title) as c from names inner join director_mapping on director_mapping.name_id=names.id inner join movie on movie.id=director_mapping.movie_id group by names.name having c>5;

-- 3. Retrieve all movies that have at least one review.
select movie.title,count(ratings.total_votes) as c from movie inner join ratings on movie.id=ratings.movie_id group by movie.title having c>=1;

-- 4. Get a list of all movies along with their production company names.
select m.title,n.production_company from movie as m inner join movie as n on m.id=n.id;

-- 5. Fetch all movies with their country of origin.
select m.title,n.country from movie as m inner join movie as n on m.id=n.id;


-- Left Joins

-- 1. List all movies, even those without ratings.
select movie.title,ratings.avg_rating from movie left outer join ratings on movie.id=ratings.movie_id;

-- 2. Retrieve all actors, even those who have not acted in any movie.
select names.name from names inner join role_mapping on names.id=role_mapping.name_id left outer join movie on movie.id=role_mapping.movie_id where role_mapping.category="actor";

-- 3. Get all directors, including those who haven't directed any movies.
select names.name from names inner join role_mapping on names.id=role_mapping.name_id left outer join movie on movie.id=role_mapping.movie_id;


-- Right Joins

-- 1. Find all movies and ensure that genres are included even if no movies belong to that genre.
select movie.title,genre.genre from movie right outer join genre on movie.id=genre.movie_id;

-- 2. Get all movies and ensure all countries are listed even if no movie was produced there.
select m.title,n.country from movie as m right outer join movie as n on m.id=n.id;

-- 3. Retrieve all writers and ensure that movies written by them (if any) are included.
select movie.title,ratings.avg_rating from movie right outer join ratings on movie.id=ratings.movie_id;

-- 4. List all reviews and the movies they belong to, including reviews without associated movies.
select m.title,n.production_company from movie as m right outer join movie as n on m.id=n.id;

-- 5. Fetch all production companies and the movies they have produced, even if no movies are linked to some companies.
-- No Answer Yet


-- Complex Joins (Multiple Joins)

-- 1. Retrieve the top 5 highest-rated movies along with their directors and genres.
select movie.title,names.name,genre.genre,ratings.avg_rating from names inner join director_mapping on names.id=director_mapping.name_id inner join movie on movie.id=director_mapping.movie_id inner join ratings on ratings.movie_id=movie.id inner join genre on genre.movie_id=movie.id order by ratings.avg_ratings desc limit 5;

-- 2. Find actors who have worked with the same director in at least 3 movies.
select s.director,t.actor,count(*) as m from (select names.name as director,director_mapping.movie_id as id from names inner join director_mapping on names.id=director_mapping.name_id) as s inner join (select names.name as actor,role_mapping.movie_id as id from names inner join role_mapping on names.id=role_mapping.name_id inner join movie on movie.id=role_mapping.movie_id) as t on s.id=t.id group by s.director,t.actor having m>=3;

-- 3. Get a list of all movies that have both an actor and a writer in common.
-- No Answer Yet

-- 4. List all directors and the actors they have worked with, even if they directed only one movie.
select s.director,t.actor,count(*) from (select names.name as director,director_mapping.movie_id as id from names inner join director_mapping on names.id=director_mapping.name_id) as s inner join (select names.name as actor,role_mapping.movie_id as id from names inner join role_mapping on names.id=role_mapping.name_id inner join movie on movie.id=role_mapping.movie_id) as t on s.id=t.id group by s.director,t.actor having count(*)>=1;

-- 5. Retrieve all movies with their genres, directors, in one query.
select movie.title,names.name,genre.genre from names inner join director_mapping on names.id=director_mapping.name_id inner join movie on movie.id=director_mapping.movie_id inner join genre on movie.id=genre.movie_id;


-- Partition By and Window Function Questions

-- 1. Retrieve the top 5 highest-rated movies per genre using RANK().
select a.title,a.avg_rating from (select movie.title,ratings.avg_rating,rank() over(partition by genre.genre order by ratings.avg_rating desc) as pro from movie inner join ratings on movie.id=ratings.movie_id inner join genre on movie.id=genre.movie_id) as a where a.pro<=5;

-- 2. Find the cumulative revenue for each movie ordered by release date using SUM() OVER().
select date_published,title,sum(duration) over(order by date_published,title) as revenue from movie;

-- 3. Get the previous movie rating for each movie using LAG().
select movie.title,ratings.avg_rating,lag(avg_rating) over(partition by movie.title order by movie.date_published) as previous from movie inner join ratings on movie.id=ratings.movie_id;

-- 4. Get the next movie rating for each movie using LEAD().
select movie.title,ratings.avg_rating,lead(avg_rating) over(order by movie.date_published) as next from movie inner join ratings on movie.id=ratings.movie_id;

-- 5. Retrieve the average movie rating per genre using AVG() OVER(PARTITION BY genre).
select movie.title,genre.genre,avg(ratings.avg_rating) over(partition by genre.genre) as m from movie inner join ratings on movie.id=ratings.movie_id inner join genre on movie.id=genre.movie_id;

-- 6. Rank movies within each genre based on IMDb rating using DENSE_RANK().
select movie.title,genre.genre,ratings.avg_rating,dense_rank() over(partition by genre.genre order by ratings.avg_rating) as d from movie inner join ratings on movie.id=ratings.movie_id inner join genre on movie.id=genre.movie_id;

-- 7. Find the running total of box office earnings for movies released after 2000.
select title,year,worlwide_gross_income from movie where year>2000 order by year;

-- 8. Find the first and last movie released for each director using FIRST_VALUE() and LAST_VALUE().
select names.name,movie.title,first_value(movie.title) over(partition by names.name) as first,last_value(movie.title) over(partition by names.name) as last from movie inner join director_mapping on movie.id=director_mapping.movie_id inner join names on names.id=director_mapping.name_id;

-- 9. Compute the difference in box office earnings between each movie and the previous one for a director.
select movie.title,movie.date_published,(lag(movie.duration) over(partition by movie.date_published))-movie.duration from movie;

-- 10. Find the total number of movies each actor has acted in using COUNT() OVER(PARTITION BY actor_name).
select names.name,movie.title,count(movie.title) over(partition by names.name) as noofmovies from movie inner join role_mapping on movie.id=role_mapping.movie_id inner join names on names.id=role_mapping.name_id where role_mapping.category="actor";
