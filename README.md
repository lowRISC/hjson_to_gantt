# hjson_to_gantt.py
## Introduction
`hjson_to_gantt` is a simple script with a very simple aim: produce a Gantt 
chart based on [(H)JSON input](http://hjson.org/). It is built upon the excellent 
[python-gantt](http://xael.org/pages/python-gantt-en.html) library.

## Dependencies
`pip install gantt fuzzyfinder hjson`

## Example

    {
      projects: [
        {
          name: Project Alpha
          color: green
        }
      ]

      tasks: [
        {
          name: Design widget
          begin: 2016-10-14
          duration: 7,
          people: Farquaad
          project: alpha
        }
        {
          name: Set up widget production line
          begin: 2016-10-19
          duration: 6
          people: Zack
          project: alpha
        }
        {
          name: Manufacture widgets
          duration: 7
          people: Carrie
          deps: ["design widget", "widget prod line"]
          project: alpha
        }
      ]

      milestones: [
        {
          name: Widgets start shipping
          start: 2016-10-30
          deps: ["mftr widgets"]
          project: alpha
        }
      ]
    }

You'll note that for specifying dependencies and projects, the full name 
doesn't need to be used. Thanks to [fuzzy 
matching](https://github.com/amjith/fuzzyfinder), you can just specify e.g.
"mftr widgets" and have it match "Manufacture widgets".

To try out the example:

`./hjson_to_gantt --begin-date 2016-10-10 --end-date 2016-11-13 example.hjson --name example`

This will output `example_daily.svg` and `example_weekly.svg`.

A PNG rendering of [`example_daily.svg`](example_daily.svg) is shown below (Github doesn't allow embedded SVG):

![Example Gantt chart](example_daily.png?raw=true)

## License
This script is released under the terms of the MIT license (see LICENSE).
However, note that the python-gantt library this script uses is GPLv3+ meaning
the combination is effectively GPLv3+.
