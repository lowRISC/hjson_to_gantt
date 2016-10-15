#!/usr/bin/env python3
# Part of hjson_to_gantt http://github.com/lowRISC/hjson_to_gantt
#
# See LICENSE file for copyright and license details

import datetime
import argparse
from fuzzyfinder import fuzzyfinder
import gantt
import hjson

today = datetime.datetime.today()


def date_to_str(d):
    return datetime.datetime.strftime(d, "%Y-%m-%d")

# Option parsing
parser = argparse.ArgumentParser(
    formatter_class=argparse.ArgumentDefaultsHelpFormatter)
parser.add_argument('infile', help='(H)JSon file containing Gantt data')
parser.add_argument(
    '--begin-date', help='YYYY-MM-DD date to start Gantt chart at', default=date_to_str(today))
parser.add_argument('--end-date', help='YYYY-MM-DD date to end Gantt chart on',
                    default=date_to_str(today + datetime.timedelta(days=180)))
parser.add_argument('--name', help='Base name of SVG file to output',
                    default='gantt')
args = parser.parse_args()

top_project = gantt.Project()

json = hjson.load(open(args.infile))

# These dictionaries are always indexed by lower-cased keys
people = {}
tasks = {}
projects = {}


def ensure_list(obj):
    if isinstance(obj, str):
        return [obj]
    return obj


def parse_date(s):
    if s:
        return datetime.datetime.strptime(s, '%Y-%m-%d').date()


def parse_duration(s):
    if s:
        return int(s)


def parse_people(obj):
    if not obj:
        return
    lst = ensure_list(obj)
    ret = []
    for name in lst:
        if not people.get(name.lower()):
            people[name.lower()] = gantt.Resource(name)
        ret.append(people[name.lower()])
    return ret


def parse_deps(obj):
    if not obj:
        return
    ret = []
    deps = ensure_list(obj)
    for dep in deps:
        matches = list(fuzzyfinder(dep.lower(), tasks.keys()))
        if not matches:
            raise ValueError('Could not find a task matching "{}"'.format(dep))
        if len(matches) > 1:
            print('WARNING: found multiple matches for task "{}": {}'.format(
                dep, matches))
        task_name = matches[0]
        ret.append(tasks[task_name])
    return ret


def parse_project(s):
    matches = list(fuzzyfinder(s.lower(), projects.keys()))
    if not matches:
        raise ValueError('Could not find a project matching "{}"'.format(s))
    if len(matches) > 1:
        print('WARNING: found multiple matches for project "{}": {}'.format(s, matches))
    return projects[matches[0]]


# Add each project
for proj in json.get('projects'):
    if isinstance(proj, str):
        proj = {'name': proj}
    color = proj.get('color') or proj.get('colour')
    new_project = gantt.Project(name=proj['name'], color=color)
    projects[proj['name'].lower()] = new_project
    top_project.add_task(new_project)

# Add each task
for task in json.get('tasks', []):
    if not task.get('project'):
        raise ValueError('Task contains no project field: {}', format(task))
    project = parse_project(task.get('project'))
    gantt_task = gantt.Task(
        name=task['name'],
        start=parse_date(task.get('begin')),
        stop=parse_date(task.get('end')),
        duration=parse_duration(task.get('duration')),
        resources=parse_people(task.get('people')),
        depends_of=parse_deps(task.get('deps')),
        color=task.get('color') or task.get('colour')
    )
    tasks[gantt_task.name.lower()] = gantt_task
    project.add_task(gantt_task)

# Add each milestone
for ms in json.get('milestones', []):
    if not ms.get('project'):
        raise ValueError('Milestone contains no project field: {}', format(ms))
    project = parse_project(ms.get("project"))
    gantt_ms = gantt.Milestone(
        name=ms['name'],
        start=parse_date(ms.get('begin')),
        depends_of=parse_deps(ms.get('deps'))
    )
    project.add_task(gantt_ms)

# Output the resulting gantt chart
top_project.make_svg_for_tasks(filename=args.name + '_weekly.svg',
                               today=today, start=parse_date(args.begin_date),
                               end=parse_date(args.end_date),
                               scale=gantt.DRAW_WITH_WEEKLY_SCALE)
top_project.make_svg_for_tasks(filename=args.name + '_daily.svg',
                               today=today, start=parse_date(args.begin_date),
                               end=parse_date(args.end_date))
