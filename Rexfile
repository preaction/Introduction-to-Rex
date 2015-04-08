
group ops_prd => 'scsqhdapap0[1..4]';
group ops_uat => 'ltwqhdacap0[1..4]';
group ops_dev => 'scsqhdadap0[1..2]';

environment ops_dev => sub {
    group all => 'scsqhdadap0[1..2]';
    group reuters => 'scsqhdadap02';
    group db => 'scsqhdadap02';
};

environment ops_prd => sub {
    group all => 'scsqhdapap0[1..4]';
    group reuters => 'scsqhdapap04';
    group db => 'scsqhdapap01';
};

desc 'Check the disk free on the relevant drives';
task 'df' => sub {
    say scalar run 'df -H /usr*';
};

