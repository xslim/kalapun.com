---
title: "PHP to AD LDAP connection"
tags: [code, php]
---

A short note to myself on how to do this

``` php
<?php
if (!$this->ds) {
	$this->ds = ldap_connect($host);
	if ($this->ds) {
		ldap_set_option($this->ds, LDAP_OPT_PROTOCOL_VERSION, 3);
		ldap_set_option($this->ds, LDAP_OPT_REFERRALS, 0);
		if (!@ldap_bind($this->ds, $login, $passwd)) {
			return false;
		}
		return $this;
	} else {
		return false;
	}
}
```





``` php
public function searchUser($s) {
	$attributes = array('sAMAccountName', 'cn');
	$filter = '(&(objectCategory=user)(displayName=*'.$s.'*))';
	$sr = ldap_search($this->ds, $this->dn, $filter, $attributes);
	$num_enntries = ldap_count_entries($this->ds, $sr);
	$users = array();
	if ($num_enntries > 0) {
		$info = ldap_get_entries($this->ds, $sr);
		foreach($info as $u) {
			if (!empty($u['cn'][0]))
			$users[] = array(
					'login' => $u['samaccountname'][0],
					'name'  => $u['cn'][0],
				);
		}
		return $users;
	} else {
		return false;
	}
}

 include("pChart/pData.class");
include("pChart/pChart.class");

// Dataset definition
$DataSet = new pData;
$DataSet->AddPoint($data,"Serie1");
$DataSet->AddPoint($legend,"Serie2");
$DataSet->AddAllSeries();
$DataSet->SetAbsciseLabelSerie("Serie2");

// Initialise the graph
$graph = new pChart(720,350);
$graph->loadColorPalette('hard_bright.txt');

// Draw the pie chart
$graph->setFontProperties("Fonts/tahoma.ttf",12);
$graph->drawTitle(50,20,'Top 20 Incominf for '.$date ,0,0,0);  

$graph->setFontProperties("Fonts/tahoma.ttf",9);

$graph->drawPieGraph($DataSet->GetData(),$DataSet->GetDataDescription(),210,160,180,PIE_PERCENTAGE,TRUE,50,20,5);
$graph->drawPieLegend(420,5,$DataSet->GetData(),$DataSet->GetDataDescription(),250,250,250);

$graph->Stroke();
```
