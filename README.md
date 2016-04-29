# project for class in python that goes through movie ratings and soundtracks and creates 2 lists for movies you are comparing based on 
# movie ratings and % of movie taken up by soundtrack

import itunes
import requests
import json
import test106 as test 

def get_data_from_itunes(movie_title):
	resp = itunes.search(query = movie_title + ' sound track')
	return resp

def get_data_from_OMDB(movie_title):
	params = {'type': 'movie', 't': movie_title, 'plot':'short'}
	resp = requests.get('http://www.omdbapi.com/?', params = params)
	jdata = json.loads(resp.text)
	return jdata

class Movie:
	def __init__(self, OMDB_data, itunes_data):
		self.OMDB_data = OMDB_data
		self.itunes_data = itunes_data
		self.title = str(self.OMDB_data['Title'])

	#returns the score from rotten tomatoes that the movie recieved
	def get_imdb_rating(self):
		return self.OMDB_data['imdbRating']

	#returns the percentage of the movie taken up by the soundtrack
	def get_percent_music(self):
		album = [song.get_duration() for song in self.itunes_data]
		t = 0
		for time in album:
			t += time
		runtime = int((self.OMDB_data['Runtime']).split()[0]) * 60
		return str((float)(t)/(float)(runtime)) + " percent of the movie is taken up by the soundtrack"

	def get_soundtrack(self):
		album_name = self.itunes_data[0].get_albums()
		songs = self.itunes_data
		return "Sound Track: \n" + str(album_name) + '- ' + str(songs)[1:-1]

	def get_plot(self):
		return self.OMDB_data['Plot']

	def __str__(self):
		soundtrack_name = self.itunes_data[0].get_albums()
		release_year = str(self.OMDB_data['Year'])
		return "Title: " + self.title + ", Released in: " + release_year + ", SoundTrack Album Name: " + str(soundtrack_name)

#tests
itunes_data_test = get_data_from_itunes('good will hunting')
OMDB_data_test = get_data_from_OMDB('good will hunting')

good_will_hunting = Movie(OMDB_data_test,itunes_data_test)
test.testEqual(str(good_will_hunting.get_plot()),'Will Hunting, a janitor at M.I.T., has a gift for mathematics, but needs help from a psychologist to find direction in his life.')
test.testEqual(good_will_hunting.get_soundtrack(),"""Sound Track: 
Good Will Hunting (Music From the Miramax Motion Picture)- Miss Misery, Baker Street (Edit), Between the Bars, Angeles, Say Yes, Weepy Donuts (Instrumental), Fisherman's Blues, No Name #3, Will Hunting (Main Titles) [Instrumental], Somebody's Baby, How Can You Mend a Broken Heart, Boys Better, As the Rain, Between the Bars (Orchestral), Why Do I Lie? (Remix), Theme from "Good Will Hunting", Baker Street (From "Good Will Hunting"), Baker Street (Good Will Hunting)""")
test.testEqual(good_will_hunting.get_percent_music(),'0.53639021164 percent of the movie is taken up by the soundtrack')


#involcation of the methods and functions
movie = raw_input("Enter a movie you'd like to know a bit more about or q if you just want to compare a few: ")
if movie != 'q':
	itunes_data = get_data_from_itunes(movie)
	OMDB_data = get_data_from_OMDB(movie)
	m = Movie(OMDB_data,itunes_data)
	print m
	print m.get_plot()
	print m.get_imdb_rating()
	print m.get_percent_music()
	print m.get_soundtrack()

print "Enter some movies that you'd like to compare, enter q when you would like to stop: "
i = 1
x = raw_input(str(i) + ". ")
list_of_movies = []
while x != 'q':
	list_of_movies.append(x)
	i += 1
	x = raw_input(str(i) + ". ")

def compare_movies(list_of_movies):
	movie_objects = []
	for m in list_of_movies:
		movie_objects.append(Movie(get_data_from_OMDB(m),get_data_from_itunes(m)))
	imdb_list = []

	for movie in movie_objects:
		d = {}
		d['Title'] = movie.title
		d['IMDB Rating'] = movie.get_imdb_rating()
		d['Percentage Music'] = movie.get_percent_music()
		imdb_list.append(d)

	by_imdb = sorted(imdb_list, key = lambda k: k['IMDB Rating'])
	by_percent_music = sorted(imdb_list, key = lambda k: k['Percentage Music'])
	
	return "Movies: " + str(list_of_movies) +  "In order of their rating on IMDB: \n" + str(by_imdb) + "\n\n In order of the percentage of the movie taken up by the sound track: \n" + str(by_percent_music)


print compare_movies(list_of_movies)
