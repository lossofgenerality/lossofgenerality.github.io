<?php

use PHPUnit\Framework\TestCase;

require_once __DIR__ . '/../vendor/autoload.php';

class BasicTest extends TestCase {

    public function test_image_exists() {
        $this->assertTrue(\CCN\Util::image_exists('Leaving_ruledsurfaces_2192838862.PNG', 'data'));
    }

    public function test_mergejson() {
        $this->assertEquals(2, count(\CCN\Util::mergejson([], 'data/module.json')));
    }

    public function test_validate_file() {
        $this->assertFalse(\CCN\Util::validate_file('bad.file', 'data'));

        $this->assertFalse(\CCN\Util::validate_file(json_decode('{"image": "bad.file.GIF"}'), 'data'));

        $this->assertTrue(\CCN\Util::validate_file('good_agif.gif', 'data'));

        $this->assertTrue(\CCN\Util::validate_file(json_decode('{"image": "good_agif.gif"}'), 'data'));
        $this->assertTrue(\CCN\Util::validate_file(json_decode('{"image": "good_agif.gif", "optimize": false}'), 'data'));
        $this->assertTrue(\CCN\Util::validate_file(json_decode('{"image": "good_agif.gif", "optimize": true}'), 'data'));
    }

    public function test_convert_file() {
        $this->assertEquals('image.png', \CCN\Util::convert_file('image.png', 'data'));
        $this->assertEquals('image.SVG', \CCN\Util::convert_file('image.eps', 'data'));
    }

    public function test_process_json() {
        $json = json_decode(file_get_contents('data/process_json_blank.json'));
        $module = CCN\Util::process_json($json, 'data');
        $this->assertEquals(0, count($module));

        $json = json_decode(file_get_contents('data/process_json_simple.json'));
        $module = CCN\Util::process_json($json, 'data');
        $this->assertEquals(1, count($module));

        $json = json_decode(file_get_contents('data/process_json_eps.json'));
        $module = CCN\Util::process_json($json, 'data');
        $this->assertEquals(1, count($module));
        $this->assertEquals(1, count($module[0]));
        $this->assertEquals($module[0][0]->question, "quiz3_ARAB_9935314434.SVG");
        $this->assertEquals($module[0][0]->choices[0], "quiz3_ARAB_9935314434.SVG");
    }

}
