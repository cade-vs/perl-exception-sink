# NAME

Exception::Sink - general purpose compact exception handling.

# SYNOPSIS

    use Exception::Sink;

    eval
      {
      eval
        {
        # use one of the following for testing:
        sink 'BIG: this has no ID, should be surfaced by the global handler';
        sink 'USUAL: this has no ID, should be surfaced by the local handler';
        sink 'FATAL: EXAMPLE: fatal exception with ID "EXAMPLE", will not be handled';
        sink 'STRANGE: EXAMPLE: fatal exception with ID "EXAMPLE", will not be handled';
        };
      if( surface 'USUAL' ) # local handler
        {
        print "surface USUAL, handled\n";
        # handle 'USUAL' exceptions here, not 'BIG' ones
        }
      else
        {
        dive();
        }
      };
    dive if surface qw( FATAL STRANGE ); # avoid global handler
    if( surface '*' ) # global handler
      {
      print "surface *, handled\n";
      # will handle all exceptions, including our 'BIG' one
      # if we don't want to handle, we can still dive forward:
      dive(); # this is the last handler so diving here will stop the program
      }
    # only FATAL:EXAMPLE will reach here but will not be reported since simple
    # hashrefs has no stringification method

# FUNCTIONS

## sink($)

    sink() gets only one argument, string with format:

     "CLASS: ID: description"
     "CLASS: description"
     "description"

    exception will have accordingly:

     CLASS and ID
     CLASS only
     CLASS will be 'SINK'

    then it will throw (sink/dive) an exception hash ref.

## surface(@)

    surface() will return true if argument list matches currently
    sinking exception:

    if( surface qw( BIG_ONE FATAL TESTING ) )
      {
      # handle one of BIG_ONE FATAL TESTING exception classes
      }
    else
      {
      # if not matched try to dive() (resink) below...
      dive();
      }

## surface2(@)

    same as surface but will dive() if exception class has not been matched.
    i.e those are equal:

    if( surface( ... ) ) { handle } else { dive }

    handle if surface2( ... )

## dive()

    will continue/propagate currently sinking exception

## boom($)

    special version of sink() it will always has class 'BOOM' and will has
    full stack trace with pid information added to the sink() description text.

## get\_stack\_trace()

    this is utility function, which returns array with formatted stack trace
    lines, containing package, function names, file with line number. it can
    be called at any time, usually for debug purpose. it is not exported by
    default!

# EXCEPTION STRUCTURE

    Executing this:

    sink "SINK: UNKNWON: here is the text of the exception";

    will create this exception data hash:

    $@ = {
            'CLASS'   => 'SINK',      # exception class, used by surface()
            'ID'      => 'UNKNOWN',   # this is optional error-id
            'FILE'    => 'Sink.pm',   # file where sink started
            'LINE'    => 87           # line where sink started
            'PACKAGE' => 'Sink',      # package where sink started
            'MSG'     => 'here is the text of the exception',
         };

    'CLASS' is used by surface() to filter which exceptions should be handled.
    'ID'    is used only by the exception handling code to figure what exactly
            has happened.

    The other attributes are for information puproses (debugging).

# NOTES

    You may freely use die() instead of sink(). The following surface()/dive()
    will resink into hash reference.

    surface() will not dive/sink more if exception did not match class list.
    If you want surface() to handle class or otherwise to continue dive/sink,
    you should use surface2() instead:

    eval
      {
      eval
        {
        sink "TESTING: testing resink/dive surface2()";
        };
      if( surface2 'BIG_ONE' )
        {
        # only BIG_ONE exception will be handled here,
        # all the rest will dive/resink below...
        }
      };
    # TESTING exception will reach here

    If you do not want to autoimport all functions:

    use Exception::Sink qw( :none )

    If you want to use only surface() (probably with die() instead of sink() ):

    use Exception::Sink qw( :none surface )

# TODO

    (more docs)

# GITHUB REPOSITORY

    git@github.com:cade-vs/perl-exception-sink.git
    

    git clone git://github.com/cade-vs/perl-exception-sink.git
    

# AUTHOR

    Vladi Belperchinov-Shabanski "Cade"

    <cade@biscom.net> <cade@datamax.bg> <cade@cpan.org>

    http://cade.datamax.bg
