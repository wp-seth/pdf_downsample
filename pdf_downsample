#!/usr/bin/perl
# tab-size: 2

use strict;
use warnings;
use Getopt::Long qw(:config bundling);
use Pod::Usage;

$main::VERSION = '0.1.20141022';

sub syntaxCheck{
	my %params = ( # default cli params
		'dpi'         => 72,  # target dpi
		'force'       => 0,   # force
		'infile'      => '',  # infile
		'outfile'     => '-', # outfile
		'threshold'   => 1.0, # threshold
		'verbose'     => 1,   # trace; grade of verbosity
		'version'     => 0,   # diplay version and exit
	);
	GetOptions(\%params,
		"dpi|d=i",
		"force|f",
		"threshold|t=i",
		"silent|quiet|q" => sub { $params{'verbose'} = 0;},
		"very-verbose" => sub { $params{'verbose'} = 2;},
		"verbose|v:+",
		"version|V" => sub { Getopt::Long::VersionMessage();}, # auto_version will not auto make use of 'V'
		"help|?|h" => sub { Getopt::Long::HelpMessage(-verbose=>99, -sections=>"NAME|SYNOPSIS");}, # auto_help will not auto make use of 'h'
		"man" => sub { pod2usage(-exitval=>0, -verbose=>2);},
	) or pod2usage(-exitval=>2);
	$params{'verbose'} = 1 unless exists $params{'verbose'};
	# additional params
	my @additional_params = (1,2); # number of additional params (min, max);
	if(@ARGV<$additional_params[0] or ($additional_params[1]!=-1 and @ARGV>$additional_params[1])){
		if($additional_params[0]==$additional_params[1]){
			print "error: number of arguments must be exactly $additional_params[0], but is ".(0+@ARGV).".\n";
		}else{
			print "error: number of arguments must be at least $additional_params[0] and at most ".($additional_params[1] == -1 ? 'inf' : $additional_params[1]).", but is ".(0+@ARGV).".\n";
		}
		pod2usage(-exitval=>2);
	}
	$params{'infile'}  = $ARGV[0];
	die "error: file '$params{'infile'}' not found.\n" unless -e $params{'infile'};
	if(defined $ARGV[1]){
		$params{'outfile'} = $ARGV[1] ;
		die "error: file '$params{'outfile'}' exists already. use param --force to force overwriting.\n" if -e $params{'outfile'} and $params{'force'}==0;
		print "file '$params{'outfile'}' will be overwritten.\n" if -e $params{'outfile'} and $params{'force'}==1 and $params{'verbose'}>1;
	}
	return \%params;
}

my $params = syntaxCheck(@_);
my $gsverbose = $params->{'verbose'} <= 1 ? '-q ' : '';
my $cmd = 'gs '.$gsverbose.
	'-dNOPAUSE -dBATCH -dSAFER '.
	#'-dCompatibilityLevel=1.4 '.
	#'-dPDFSETTINGS=/screen '.
	#'-dEmbedAllFonts=true '.
	#'-dSubsetFonts=true '.
  "-o '".$params->{'outfile'}."' ".
  '-sDEVICE=pdfwrite '.
  '-dDownsampleColorImages=true '.
  '-dDownsampleGrayImages=true '.
  '-dDownsampleMonoImages=true '.
	#'-dColorImageDownsampleType=/Bicubic '.
  '-dColorImageResolution='.$params->{'dpi'}.' '.
	#'-dGrayImageDownsampleType=/Bicubic '.
  '-dGrayImageResolution='.$params->{'dpi'}.' '.
	#'-dMonoImageDownsampleType=/Bicubic '.
  '-dMonoImageResolution='.$params->{'dpi'}.' '.
  '-dColorImageDownsampleThreshold='.$params->{'threshold'}.' '.
  '-dGrayImageDownsampleThreshold='.$params->{'threshold'}.' '.
  '-dMonoImageDownsampleThreshold='.$params->{'threshold'}.' '.
  "'".$params->{'infile'}."'";
print "command: $cmd\n" if $params->{'verbose'} > 0;
my $errlvl = system($cmd);
if($errlvl!=0){
	print "error: gs command failed. return code was $errlvl.\n" if $params->{'verbose'} > 0;
	exit 3;
}else{
	my $old_file_size = (stat $params->{'infile'})[7];
	my $new_file_size = (stat $params->{'outfile'})[7];
	if($old_file_size == $new_file_size){
		print "notice: new file is of same size as old file.\n" if $params->{'verbose'} > 0;
		exit 1;
	}elsif($old_file_size < $new_file_size){
		print "notice: new file is even larger than old file.\n" if $params->{'verbose'} > 0;
		exit 2;
	}
	#else{ exit 0;}
}

__END__

=head1 NAME

pdf_downsample downsamples all images in a given pdf file to a given resolution 
using ghostscript.

=head1 DESCRIPTION

this program uses ghostscript to let you downsample all images in a pdf to a 
resolution given by you. 

surely this is not the first script for that job, e.g.
 http://www.alfredklomp.com/programming/shrinkpdf/
is an older example. However, the aim of this script is, to let you choose the 
resolution.

=head1 SYNOPSIS

pdf_downsample infile outfile [options]

or

pdf_downsample [options] infile outfile

  infile                 original pdf file
  outfile                new output pdf file (default: write to stdout)
  -d, --dpi              set target resolution in dpi (default=72)
  -t, --threshold        set threshold scale of images to downsample (default=1.0)
  -f, --force            overwrite existing files

meta:

  -V, --version          display version and exit.
  -h, --help             display brief help
      --man              display long help (man page)
  -q, --silent           same as --verbose=0
  -v, --verbose          same as --verbose=1 (default)
  -vv,--very-verbose     same as --verbose=2
  -v, --verbose=x        grade of verbosity
                          x=0: no output
                          x=1: default output
                          x=2: much output

some examples:

  pdf_downsample largefile.pdf smallfile.pdf
    downsamples all images in largefile.pdf to default resolution and writes result
    to target file smallfile.pdf
  
  pdf_downsample --dpi=100 largefile.pdf smallfile.pdf
    downsamples all images in largefile.pdf to 100dpi resolution and writes result 
    to target file smallfile.pdf

=head1 OPTIONS

=over 8

=item B<infile>

some pdf file that contains large images you want to downsample.

=item B<outfile>

name of the resulting pdf file. use '-' for writing the result to stdout. 
(default: '-')

=item B<--dpi>=I<resolution>, B<-d> I<resolution>

set resolution in result file to I<resolution> dpi. (default: I<resolution> = 72)

to be more precise, this will set the gs (ghostscript) parameters
 -dColorImageResolution=I<resolution>
 -dGrayImageResolution=I<resolution>
 -dMonoImageResolution=I<resolution>

=item B<--force>, B<-f>

overwrite existing files. (default = don't overwrite)

=item B<--threshold>=I<scale>, B<-t> I<scale>

set threshold of images that shall be downsampled by choosing a scaling factor of 
the I<resolution> set via B<--dpi>. all images with resolution higher than 
I<resolution> x I<scale> dpi will be downsampled. (default: I<scale> = 1.0)

examples:

if --dpi is set to 100, then I<scale>=1.0 means that all images with resolution 
higher than 100dpi will be downsampled.

if --dpi is set to 100, then I<scale>=1.5 means that all images with resolution 
higher than 150dpi will be downsampled.

to be more precise, this parameter will set the gs (ghostscript) parameters
 -dColorImageDownsampleThreshold=I<scale>
 -dGrayImageDownsampleThreshold=I<scale>
 -dMonoImageDownsampleThreshold=I<scale>

=item B<--version>, B<-V>

prints version and exits.

=item B<--help>, B<-h>, B<-?>

prints a brief help message and exits.

=item B<--man>

prints the manual page and exits.

=item B<--verbose>=I<number>, B<-v> I<number>

set grade of verbosity to I<number>. if I<number>==0 then no output
will be given, except hard errors. the higher I<number> is, the more 
output will be printed. default: I<number> = 1.

=item B<--silent, --quiet, -q>

same as B<--verbose=0>.

=item B<--very-verbose, -vv>

same as B<--verbose=2>. you may use B<-vvv> for B<--verbose=3> a.s.o.

=item B<--verbose, -v>

same as B<--verbose=1>.

=back

=head1 LICENCE

Copyright (c) 2014, seth
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

originally written by seth (see https://github.com/wp-seth/pdf_downsample)

=cut

