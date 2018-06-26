<?php

use PHPUnit\Framework\TestCase;

require_once __DIR__ . '/../vendor/autoload.php';

class AGifTest extends TestCase {

    public function test_good_gif() {
        $this->assertTrue(\CCN\Util::isValidAnimatedGif('good_agif.gif', 'data'));
    }

    public function test_bad_gif() {
        $this->assertFalse(\CCN\Util::isValidAnimatedGif('bad_agif.gif', 'data'));
    }

    public function test_simple_gif() {
        $this->assertTrue(\CCN\Util::isValidAnimatedGif('simple.gif', 'data'));
    }

    public function test_non_gif() {
        $this->assertTrue(\CCN\Util::isValidAnimatedGif('Leaving_ruledsurfaces_2192838862.PNG', 'data'));
    }

    public function test_fixgif_good() {
        array_map('unlink', glob("path/to/temp/*"));
        \CCN\Util::fixAnimatedGif('bad_agif.gif', 'tmp/animation.gif', 'data');
        $this->assertTrue(filesize('data/tmp/animation.gif') > 0);
    }

    public function test_fixgif_nonagif() {
        array_map('unlink', glob("path/to/temp/*"));
        \CCN\Util::fixAnimatedGif('simple.gif', 'tmp/animation.gif', 'data');
        $this->assertTrue(filesize('data/tmp/animation.gif') > 0);
    }

}
