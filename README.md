osm_visualization
=================
### Download the OSM files 
Download the daily improvement files in OSM from: http://planet.openstreetmap.org/replication/day/000/000/

https://github.com/oeon/osm_visualization/blob/master/data/retrieve-day

Run:

`$ ./retrieve-day arg1 arg2`

Where:

- arg1 is: 01/09/2014 = 484 (start file)
- arg2 is: 04/09/2014 = 574 (end file)

The Script is based in : https://github.com/ericfischer/ebola/blob/master/retrieve-hourly

Example:

`~/osm_visualization/data$ ./retrieve-day 448 574`

### Convert data to Geojson file

we use: https://github.com/oeon/osm_visualization/blob/master/data/get-edits:
modified from: https://github.com/ericfischer/ebola/blob/master/get-mamou-edits

Run: 

`$ ./get-edits file minlat minlon maxlat maxlon > newfile.geojson`
 
Example bounding box from San Francisco:

    $minlat = 37.716045;
    $minlon = -122.51781;
    $maxlat = 37.817006;
    $maxlon = -122.34924;

Process a file
    
Example: 

`~/osm_visualization/data$ ./get-edits 485.osc.gz 37.716045 -122.51781 37.817006 -122.34924 > sf485.geojson`

It is possible to execute all the files:

Run:

`$ ./process_all start_file end_file minlat minlon maxlat maxlon`

Example: 

` ~/osm_visualization/data$ ./process_all 484 574 37.716045 -122.51781 37.817006 -122.34924`

It will take a while depending on the number of files.

If you want to process the file for some especific user use the file, you have to edit the file: https://github.com/oeon/osm_visualization/blob/master/data/get-edits-by-users, 
specifically the line https://github.com/oeon/osm_visualization/blob/master/data/get-edits-by-users#L69 and add more users, then 

Run:

`$ ./get-edits file minlat minlon maxlat maxlon > newfile.geojson`

Example:

`~/osm_visualization/data$ ./get-edits-by-users 499.osc.gz 37.716045 -122.51781 37.817006 -122.34924 > sf499-users.geojson`

Example processing for all users:

`~/osm_visualization/data$ ./process_all_users 1 776 34.89752 -121.34792 35.79522 -119.47262`

### Creating png files

 You need to install [Tilemill](https://www.mapbox.com/tilemill), and  clone [Projectmill](https://github.com/mapbox/projectmill)

`git clone https://github.com/mapbox/projectmill.git`

#### Get a background PNG File:

In this case we need a satellite image. We use Eric Fischer's project [tile-stitch](https://github.com/ericfischer/tile-stitch), cloning in your machine an run:

    
`~/tile-stitch$ ./stitch -o sf.png -- 37.6787  -122.5171 37.8270 -122.3338 13 http://a.tiles.mapbox.com/v3/openstreetmap.map-4wvf9l0l/{z}/{x}/{y}.png`

Then we check the size of the image: in my case it's:  "width":1068, "height":1093

#### Create a project in Tilemill:

Create a project in Tilemill called sfbuildings: https://github.com/oeon/osm_visualization/tree/master/tilemill-project/sluosm2014

#### Configuration in Projectmill

We need to configure this file https://github.com/oeon/osm_visualization/blob/master/make-config.py, the exact lines are:

    project={
            "source": "sluosm2014", #name of projet in Tilemill
            "destination": "slo"+ str(x),
            "format": "png",
            "minzoom": 1,
            "maxzoom": 16,
            "width":1068, #width from tile-stitch image
            "height":1093, #height from tile-stitch image
            "mml": {
                    "Layer": [
                      {                                        
                                  "Datasource": {
                                  "file": "/home/ubuntu/osm_visualization/data/slo"+str(x)+".geojson"  #directory of files for process
                                }
                  
                      }
                    ],
                    "advanced": {},
                    "name": "line"+ str(x)
               
                  }
        }


Run:

    $ python config.py arg1 arg2

where:
arg1: is the number of the first file  that we dowload before.
arg2: is the number of the last file  that we dowload before.


Example:

    ruben@rub21:~/visualization/projectmill$ python config.py 484 574

and then execute:

   	ruben@rub21:~/Apps/visualization/projectmill$ ./index.js --mill  --render  -c config.json -f -t /usr/share/tilemill

After:

That files are created in: /home/ruben/Documents/MapBox/export.


### Created a GIF File

Copy and paste the satelital imagen from folder  tile-stitch to /MapBox/export and renamed the file  a  smaller number of the first file that was created by Tilemill:

my first file is called: sf484.png, I renamed the imagen file to sf483.png and then run:


Example:

	ruben@rub21:~/Documents/MapBox/export$ mogrify -format gif *.png && gifsicle *.gif > anim.gif
	ruben@rub21:~/Documents/MapBox/export$ gifsicle --loop=0 --colors 256 *.gif > anim.gif

The result is:

![](https://cloud.githubusercontent.com/assets/1152236/2662166/48d7280c-c038-11e3-94fd-05002489803d.gif)

Source for mae a gif: http://www.lcdf.org/gifsicle/man.html
