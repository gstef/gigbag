#!/usr/bin/python3
#
# Copyright (C) 2014 Grzesiek Stefanek
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License ver.2
# as published by the Free Software Foundation

import sys, string, re, getopt

def load_file (filename):
    """Returns input file as a list of strings, trimming whitespace at end"""
    try:
        infile = open(filename,'r')
    except IOError:
        print('Error: '+filename+' not found')
        sys.exit(1)
    linelist = []
    for line in infile:
        if line.isspace():
            linelist.append('\n')
        else:
            if line.endswith('\n'): 
                linelist.append( line[:-1].rstrip() )
            else:   
                linelist.append(line)           
    infile.close()
    return linelist

def save_file(filename,songbook,append):
    """Saves output and adds missing newlines"""
    if append:
        try:
            outfile = open(filename,'a')
        except IOError:
            print('Error while saving file',filename)
            sys.exit(1)
    else:
        try:
            outfile = open(filename,'w')
        except IOError:
            print('Error while saving file: ',filename)
            sys.exit(1)
    for line in songbook:
        if line.isspace():
            outfile.write('\n')
        else:
            outfile.write(line+'\n')        
    outfile.close()
    print('LaTeX source saved as ',filename)

def multiply_keywords(keywords):
    """Warning, code so crude it may give you cancer, Will be replaced ASAP"""
    output = []
    output += keywords
    # Case variations
    for word in keywords:
        temp = word.capitalize()
        if temp not in keywords:
            output.append(temp)
    for word in keywords:
        temp = word.lower()
        if temp not in keywords:
            output.append(temp)
    for word in keywords:
        temp = word.upper()
        if temp not in keywords:
            output.append(temp)
    temp_output = []
    #Dots, colons, spaces
    for word in output:
        temp = word+':'
        if temp not in temp_output:
            temp_output.append(temp)
    for word in temp_output:
        if word not in output:
            output.append(word)
    temp_output = []
    for word in output:
        temp = word+'.'
        if temp not in temp_output:
            temp_output.append(temp)
    for word in temp_output:
        if word not in output:
            output.append(word)
    temp_output = []
    for word in output:
        temp = word+' '
        if temp not in temp_output:
            temp_output.append(temp)
    for word in temp_output:
        if word not in output:
            output.append(word)
    temp_output = []
    # Numbering
    for i in range (1,10):
        for word in output:
            temp = word+str(i)
            if temp not in output:
                temp_output.append(temp)
    output += temp_output
    for word in output:
        temp = word+':'
        if temp not in temp_output:
            temp_output.append(temp)
    for word in temp_output:
        if word not in output:
            output.append(word)
    temp_output = []
    return output
    
def load_keywords (category, filename):
    """Loads keywords of a given type from a config file"""
    content = load_file(filename)
    keywords = []
    flag = False
    for line in content:
        if line[0] == '#':
            flag = False
        if flag and not line.isspace(): 
            keywords.append(line.strip())
        if line[0] == '#' and line[1:].strip() == category:
            flag = True

    return multiply_keywords(keywords)
    
def load_markers(filename):
    """Deprecated, stays just in case"""
    print('Loading markers: ',filename,' ...')
    in_file = open(filename,'r')
    markers = []
    for line in in_file:
        markers.append(line[:-1])
    in_file.close()
    return markers
    
def load_chords_definitions(filename):
    """Loads named chords"""
    print('Loading chord names...')
    in_file = open(filename,'r')
    markers = []
    for line in in_file:
        if not line.isspace():
            temp = line.split()
            markers.append([temp[0],temp[1]])
    in_file.close()
    return markers
    
def load_chords_names(filename):
    """Loads named chords and definitions"""
    print('Loading chords definitions...')
    in_file = open(filename,'r')
    markers = []
    for line in in_file:
        if not line.isspace():
            temp = line.split()
            markers.append(temp[0])
    in_file.close()
    return markers

def get_chords_names(chords_definitions):
    """Extracts chord names into a separate list"""
    output = []
    for record in chords_definitions:
        output.append(record[0])
    return output
    
def clear_double_newlines(songbook):
    """Clears any double empty lines for future parsing"""
    to_delete = []
    print('Clearing double newlines...')
    for i in range(0, len(songbook)-1):
        if songbook[i] == '\n' and songbook[i+1] == '\n':
            to_delete.append(i)
    for index in sorted(to_delete, reverse=True):
        del songbook[index]
    return songbook

def separate_markers (songbook,markers):
    """Separates markers from line if found at the beginning"""
    temp_songbook = []
    for line in songbook:
        if line.isspace():
            temp_songbook.append(line)
        else:            
            words = line.split()
            if words[0] in markers and len(words)>1:
                temp_songbook.append(words[0])
                rest = ''
                for word in words:
                    rest = rest + word + ' '
                    temp_songbook.append(rest)
            else:
                temp_songbook.append(line)
    return temp_songbook
    
def is_group_marker (line):
    """Checks if a line marks a new group of blocks (section, song)"""
    temp = line.strip()
    if line.isspace():
        return None   
    if temp[0] == '#':
        if temp[1] == '#':
            return 'song_marker'
        else:
            return 'section_marker'
    else:
        return None

def is_section_marker (line):
    """ Returns true if a line is a section marker (Deprecated)"""
    temp = line.strip() 
    if not temp.isspace():
        if len(temp)>1:
            if temp[0] == '#' and temp[1] != '#':
                return True
            else:
                return False
        else:
            return False
    else:
        return False

def is_song_marker (line):
    """ Returns true if a line is a song marker"""
    temp = line.strip() 
    if not temp.isspace():
        if len(temp)>1:
            if temp[0] == '#' and temp[1] == '#':
                return True
            else:
                return False
        else:
            return False
    else:
        return False

def unify_chords (line):
    """Changes brackets to squares if needed"""
    temp = line.strip()
    temp = temp.replace('(','[')
    temp = temp.replace(')',']')
    return temp

def strip_chords (line):
    """Prepares bracketed or TEXed chords for matching with a chord list"""
    temp = unify_chords (line)
    temp = temp.replace('[',' ')
    temp = temp.replace(']',' ')
    temp = temp.replace('\\',' ')
    return temp
    
def is_chords (line, chords):
    """Returns true if a line contains chords and chords only"""
    if line.isspace(): return False
    temp = strip_chords (line)
    words = temp.split()
    for word in words:
        if word[0] =='%':
            return False
    for word in words:
        if word not in chords:
            return False
    return True
    
def chords_at_end (line, chords):
    """Returns true if a line ends with one or more chords"""
    if line.isspace(): return False
    temp = unify_chords (line)
    words = temp.split()
    if len(words) < 2:
        return False
    if strip_chords(words[0]) in chords:
        return False
    else:
        temp_line = ''
        if strip_chords(words[-1]) in chords:
            for word in words:
                if strip_chords(word) not in chords:
                    temp_line = temp_line + ' '  + word
                else:
                    break
            if ('[' or ']') in temp_line:
                return False
            else:
                return True
            
def chords_in_line (line, chords):
    """Returns true if a line contains square brackets"""
    if ('[' or ']') in line:
        return True
    else:
        return False
    
def is_marker (line, markers):
    """Returns true if a line is a defined song section keyword"""
    if line.isspace():
        return False
    else:
        if line.strip() in markers:
            return True
        else:
            return False

def what_is (line, markers_dict,chords):
    """Identifies line content"""
    if line.isspace():
        return 'empty'
    if is_song_marker(line.strip()):
        return 'song_marker'
    # Legacy, best not touch this
    if is_section_marker(line.strip()):
        return 'song_marker'
    if is_chords(line.strip(),chords):
        return 'chords'
    if chords_at_end(line.strip(),chords):
        return 'chords_at_end'
    if chords_in_line(line.strip(),chords):
        return 'chords_in_line'
    if line.strip() in markers_dict['verse']:
        return 'verse_marker'
    if line.strip() in markers_dict['chorus']:
        return 'chorus_marker'
    if line.strip() in markers_dict['note']:
        return 'note_marker'
    if line.strip() in markers_dict['intro']:
        return 'intro_marker'
    return 'unknown'
    
def separate_blocks ( songbook, markers ):
    """Inserts a newline between blocks if missing"""
    temp = []
    for line in songbook:
        if line.strip() in markers:
            temp.append('\n')
        temp.append(line)
    return temp

def integrate_blocks ( songbook, markers ):
    """Removes a newline between markers and rest of block if unnecessary"""
    flag = False    
    new_songbook = []
    print('Removing unnecessary block separators...')
    new_songbook.append(songbook[0])
    for i,line in enumerate(songbook):
        if i > 0:
            if not (songbook[i-1].strip() in markers and line.isspace()):
                new_songbook.append(line)
        
    return new_songbook            

def add_border_newlines ( songbook ):
    """Adds a newline at beginning and end of file for easier parsing"""
    print('Adding markers at the beginning and end of file...')
    temp = songbook
    if temp[0] != '\n':
        temp.insert(0,'\n')
    if temp[-1] != 'n':
        temp.append('\n')
    return temp

def what_block ( block, markers_dict, chords ):
    """Identifies a type of a block based on content"""
    content = block['lines']
    if content == []:
        return 'empty'
    first_line = what_is(content[0], markers_dict, chords)
    if first_line == 'section_marker':
        return 'section_header'
    if first_line == 'song_marker':
        return 'song_header'
    if first_line == 'verse_marker':
        return 'default'
    if first_line == 'intro_marker':
        return 'intro'
    if first_line == 'chorus_marker':
        return 'chorus'
    if first_line == 'note_marker':
        return 'note'
    return 'default'
        
def chop_blocks ( songbook, markers_dict, chords ):
    """Cuts input stream into blocks for easier processing"""
    print('Chopping input stream...')
    block = {'block_start' : 0,
             'block_end' : 0,
             'type' : 'default',
             'lines' : [],
             'number' : 0
             }
    blocks = []
    counter = 0
    for i, line in enumerate(songbook):
        block = {'block_start' : 0,
                'block_end' : 0,
                'type' : 'default',
                'lines' : [],
                'number' : 0
             }
        if line.isspace():
            counter = counter +1
            for ii in range(i,len(songbook)):
                if not songbook[ii].isspace():
                    block['lines'].append(songbook[ii])
                if ii > i and songbook[ii].isspace():
                    break
            block['block_start'] = i
            block['block_end'] = ii
            block['number'] = counter
            block['type'] = what_block(block,markers_dict,chords)
            if block['lines'] != []:
                blocks.append(block)
    return blocks
    
def find_chords ( line ):
    """Returns a list of all bracketed chords found in a line"""
    match = []
    match = re.findall(r'\\*\[[\w+-]+\]', line)
    return match

def find_all_chords(lines,chord_definitions):
    """Returns a list of all bracketed chords found in a list of lines, with duplicates removed"""
    all_chords = []
    chords = get_chords_names(chord_definitions)
    for line in lines:
        chords_in_line = find_chords(line)
        for chord in chords_in_line:
            temp = strip_chords(chord).strip()
            if temp not in all_chords:
                all_chords.append(temp)
            if temp not in chords:
                print('Warning: unrecognized chord: ',temp)
    return all_chords
    
def tex_chorded(line):
    """Formats a line of lyrics and inline chords with LaTeX notation""" 
    temp = line.replace('[',r'\[')
    temp = temp.replace(r'\\','\\')
    return temp

def tex_instrumental(line, chords):
    """Formats a line of chords-only with LaTeX notation"""
    temp1 = strip_chords(line)
    words = temp1.split()
    temp = ''
    temp = temp + r'\nolyrics{'
    for word in words:
        if word in chords:
            chord = '\\[' + word + '] '
            temp = temp + chord
        else:
            print('Warning: unknown chord: '+word+' in line: '+line)
            chord = '\\[' + word + '] '
            temp = temp + chord
    temp = temp + '}'
    return temp
    
def tex_chords_at_end(line, chords):
    """Formats a line ending with a chord sequence with LaTeX notation"""
    words = line.split()
    lyric_line = ''
    chord_line = ''
    first_chord = 0
    for i,word in enumerate(reversed(words)):
        if word not in chords:
            first_chord = i
            break
    for word in words[:-first_chord]:
        lyric_line = lyric_line + ' ' + word
    for word in words[-first_chord:]:
        chord_line = chord_line + ' ' + word
    output_line = lyric_line + ' ' + tex_instrumental(chord_line, chords)
    return output_line
    
def insert_chordline (line1, line2):
    """Formats a line of lyrics and a line of chords above into a single line"""
    chord_list = []
    chord_line = line1.replace('\t','    ')
    lyric_line = line2.replace('\t','    ')
    chord_line += ' '
    found_chord = ''
    in_chord = False
    joined_line = ''
    for i in range(0,len(chord_line)):
        if chord_line[i] ==' ':
            in_chord = False
        if chord_line[i] != ' ' and not in_chord:
            for ii in range (i,len(chord_line)):
                if chord_line[ii] == ' ':
                    found_chord = chord_line[i:ii]
                    chord_list.append([i,found_chord])
                    break
            in_chord = True
    if len(lyric_line) > len(chord_line):
        max_len = len(lyric_line)
    else:
        max_len = len(chord_line)
    for i in range (0, max_len):
        if i < len(lyric_line):
            joined_line += lyric_line[i]
        for record in chord_list:
            if record[0] == i:
                joined_line += '['+record[1]+']'        
    return joined_line
    
def output_block_content(block,markers_dict,chords):
    """Outputs content of a block with LaTeX notation"""
    output = []
    lines_raw = block['lines']
    lines_raw.append('\n')
    pass_next = False
    for i, line in enumerate(lines_raw):
        if line == '\n':
            break
        if what_is(lines_raw[i],markers_dict,chords) == 'chords':
            if what_is(lines_raw[i+1],markers_dict,chords) == 'unknown':
                output.append(tex_chorded(insert_chordline(lines_raw[i],lines_raw[i+1])))
                pass_next = True
            else:
                output.append(tex_instrumental(lines_raw[i],chords))
        else: 
            if not pass_next:
                if what_is(lines_raw[i],markers_dict,chords) == 'chords_at_end':
                    output.append( tex_chords_at_end (lines_raw[i], chords) )
                if what_is(lines_raw[i],markers_dict,chords) == 'chords_in_line':
                    output.append(tex_chorded(lines_raw[i]))
                if what_is(lines_raw[i],markers_dict,chords) == 'unknown':
                    output.append(lines_raw[i])
            else:
                pass_next = False
    temp = []
    for line in output:
        temp.append(line.strip().replace('%',''))
    output = temp
    return output

def output_section_header(block):
    """Deprecated!"""
    output = []
    section_title = block['lines'][0]
    section_title = section_title[1:].strip()
    output.append('\\beginsection{'+section_title+'}')
    return output
    
def identify_songs (blocks):
    """ Returns pairs of first and last blocks af all songs found in a block set"""
    print('Identifying songs...')
    song_start = 0
    song_end = 0
    headers = []
    songs_table = []
    for i, block in enumerate(blocks):
        if block['type'] == 'song_header':
            headers.append(i) 
    headers.append(len(blocks))
    for i in range(0, len(headers)-1):
        songs_table.append([headers[i],headers[i+1]-1])
    return songs_table

def tex_diagrams(lines,chord_definitions):
    """Returns LaTeX code rendering chord diagrams for chords found in list of strings"""
    chords = find_all_chords(lines,chord_definitions)
    output = []
    output.append('\\musicnote{')
    for chord in chords:
        for i in range(0,len(chord_definitions)):
            if chord == chord_definitions[i][0]:
                code = chord_definitions[i][1]
        output.append('\\gtab{'+chord+'}'+'{'+code+'}')
    output.append('}')
    return output
    

def output_song (song,blocks,markers_dict,chords_definitions):
    """ Returns a ready LaTeX output for a given song"""

    # Loading chords
    chords = get_chords_names(chords_definitions)

    # Preparing defaults
    start = song[0]
    end = song[1]+1
    output = []
    first_line = ''
    first_chorus_line = ''
    artist = ''
    author = ''
    title = ''
    alt_title = ''
    
    # Song header
    for block_number in range (start, end):
        if blocks[block_number]['type']  == 'song_header':
            temp = blocks[block_number]['lines'][0]
            if '/' in temp:
                temp = temp.replace('#','')
                temp = temp.split('/')
                title = temp[0].strip()
                alt_title = temp[1].strip()
            else:
                temp = temp.replace('#','')
                title = temp.strip()
            if len(blocks[block_number]['lines']) > 1:
                artist = blocks[block_number]['lines'][1]
            if len(blocks[block_number]['lines']) > 2:
                author = blocks[block_number]['lines'][2]
    print('Processing song "',title,'"')
    output.append('\\beginsong{'+title+'}[')
    output.append('  by={'+artist+'},')
    output.append('  cr={'+author+'},')  
    output.append('  ititle={'+alt_title+'}]\n')
    
    #Song content
    for block_number in range (start, end):
        if blocks[block_number]['type'] == 'chorus':
            output.append('\\beginchorus')
            output += output_block_content(blocks[block_number],markers_dict,chords)
            output.append('\\endchorus')
            output.append('\n')
        if blocks[block_number]['type'] == 'default':
            output.append('\\beginverse')
            output += output_block_content(blocks[block_number],markers_dict,chords)
            output.append('\\endverse')
            output.append('\n')
        if blocks[block_number]['type'] == 'intro':
            output.append('\\beginverse*')
            output.append(blocks[block_number]['lines'][0])
            output += output_block_content(blocks[block_number],markers_dict,chords)
            output.append('\\endverse')
            output.append('\n')
    
    # Chord diagrams
    output += tex_diagrams(output,chords_definitions)
    
    # Finishing
    output.append('\\endsong\n')
    return output

def usage():
    """Standard help"""
    print('Usage: python3 gigbag.py [-options] <input_file> ')
    print()
    print('Options:')
    print('-o   --output      Specify output file. By defaults appends to songs.sbd if not specified')
    print('-k   --keywords    Specify keywords file. Defaults to keywords.cfg if not specified')
    print('-c   --chords      Specify chord definitions file. Defaults to chords_ukulele if not specified')
    print('-w   --overwrite   Overwrites output file instead of appending')
    
    
def main ( argv ):
    """Main routine"""
    print('Gigbag ver 0.6.1')
    
    # File defaults:
    chord_file = 'chords_ukulele'
    keywords_file = 'keywords.cfg'
    output_file = 'songs.sbd'
    append = True
    
    # Arguments handling
    try:                                
        opts, args = getopt.getopt(argv,"k:o:hc:w", ["keywords","output","help","chords","overwrite"])
    except getopt.GetoptError:
        usage()                          
        sys.exit(2) 
    try:
        input_file = args[0]
    except IndexError:
        usage()
        sys.exit(2)
    for opt, arg in opts:
        if opt in ('-h', '--help'):
            usage()
            sys.exit()
        if opt in ('-o','--output'):
            output_file = arg                  
        if opt in ("-c", "--chords"):
            chord_file = arg
        if opt in ('-w',"--overwrite"):
            append = False    
        if opt in ('-k',"--keyword"):
            keywords_file = arg
            
    # Console chatter
    print('Using input file:                    ',input_file)
    print('Using keywords file:                 ',keywords_file)
    print('Using chords file:                   ',chord_file)
    if append:
        print('Using output file in append mode:    ',output_file)
    else:
        print('Using output file in overwrite mode: ',output_file)
            
    # Preparation
    Chords_definitions = load_chords_definitions(chord_file)
    Chords = get_chords_names(Chords_definitions)
    Intro_markers = load_keywords('INSTRUMENTAL',keywords_file)
    Chorus_markers = load_keywords('CHORUS',keywords_file)
    Verse_markers = load_keywords('VERSE',keywords_file)
    Note_markers = load_keywords('COMMENT',keywords_file)
    All_markers = Intro_markers + Chorus_markers + Verse_markers + Note_markers
    Markers_dict = {
                    'intro' : Intro_markers,
                    'chorus' : Chorus_markers,
                    'verse' : Verse_markers,
                    'note' : Note_markers,
                    'all' : All_markers
                    }
    songbook = load_file (input_file)
    
    # Parsing
    songbook = clear_double_newlines(songbook)
    songbook = separate_blocks(songbook,All_markers)
    songbook = integrate_blocks(songbook,All_markers)
    songbook = separate_markers(songbook,All_markers)
    songbook = clear_double_newlines(songbook)
    songbook = add_border_newlines(songbook)
    Blocks = chop_blocks(songbook,Markers_dict,Chords)
    Songs_table = identify_songs(Blocks)
    
    # LaTeX output
    file_output = []
    for i, song in enumerate(Songs_table):
        song_content = output_song(song,Blocks,Markers_dict,Chords_definitions)
        file_output += song_content
        
    # That's it
    save_file(output_file,file_output,append)
    
if __name__ == "__main__":
    main( sys.argv[1:] )






