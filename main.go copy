package main

import (
	"net/http"
	"io/ioutil"
	"encoding/json"
	"fmt"
	"gopkg.in/mgo.v2"
	"log"
	"gopkg.in/yaml.v2"
)

type PrometheusResponse struct {
	_id    string
	Status string
	Data   Data
}

type Data struct {
	ResultType string
	Result     interface{}
}

type VectorResult struct {
	Metric Metric
	Value  Value
}

type MatrixResult struct {
	Metric Metric
	Values []Value
}

type Metric struct {
	__name__ string
	Instance string
	Job      string
}

type Value struct {
	Time   float64
	Sample int
}

type YamlFile struct {
	Metrics []string
}

func main() {

	persistMetrics()
}

func persistMetrics() {
	prometheusResponses := getNewMetrics()
	for _, prometheusResponse := range prometheusResponses {
		fmt.Println(prometheusResponse)
	}

	mongoSession := getMongoSession()

	//defer mongoSession.Close()
	error := mongoSession.DB("Test").C("Metrics").Insert(prometheusResponses[0])
	fmt.Println(error)
}

func getSavedMetrics() {
	//prometheusResponses := []PrometheusResponse
	mongoSession := getMongoSession()
	iterator := mongoSession.DB("Test").C("Metrics").Find(nil).Iter()

	for !iterator.Done() {
		var prometheusResponse interface{}
		iterator.Next(&prometheusResponse)
		//prometheusResponses = append(prometheusResponses, prometheusResponse)
		fmt.Println(prometheusResponse)
	}
}

func getMongoSession() *mgo.Session {
	mongoSession, mongoError := mgo.Dial("mongodb://0.0.0.0:8000")
	if mongoError == nil {
		//defer mongoSession.Close()
		return mongoSession
	} else {
		return nil
	}
}

func getNewMetrics() ([]PrometheusResponse) {
	var yamlFile YamlFile

	yamlFileRaw, error := ioutil.ReadFile("metrics.yaml")
	//fmt.Println(51)

	if error != nil {
		//fmt.Println(52)
		log.Printf("yamlFileRaw.Get error   #%v ", error)
	}
	//fmt.Println(yamlFileRaw)
	error = yaml.Unmarshal(yamlFileRaw, &yamlFile)
	//fmt.Println(59)
	if error != nil {
		//fmt.Println(58)
		log.Fatalf("Unmarshal: %v", error)
	}

	fmt.Println(yamlFile.Metrics)
	prometheusResponses := []PrometheusResponse{}

	for _, metric := range yamlFile.Metrics {
		url := "https://ceapps-prometheus-t17a.us-west-2.nonprod.cnqr.delivery/api/v1/query?query=" + metric
		var prometheusResponse PrometheusResponse
		body := getMetricsFromURL(url)
		json.Unmarshal(body, &prometheusResponse)
		//fmt.Println(prometheusResponse)
		prometheusResponses = append(prometheusResponses, prometheusResponse)
	}

	return prometheusResponses
}

func getMetricsFromURL(url string) []byte {
	url = "https://ceapps-prometheus-t17a.us-west-2.nonprod.cnqr.delivery/api/v1/query?query=go_gc_duration_seconds"
	//url := "http://localhost:9090/api/v1/query?query=up&time=2018-06-20T22:10:51.781Z"
	//map[metric:map[__name__:up instance:localhost:9090 job:prometheus] value:[1.529532651781e+09 1]]

	//url := "http://localhost:9090/api/v1/query_range?query=up&start=2018-06-20T21:10:51.781Z&end=2018-06-20T21:11:51.781Z&step=15s"
	//map[values:[[1.529529051781e+09 1] [1.529529066781e+09 1] [1.529529081781e+09 1] [1.529529096781e+09 1] [1.529529111781e+09 1]] metric:map[__name__:up instance:localhost:9090 job:prometheus]]
	resp, err := http.Get(url)
	/*http://localhost:9090/api/v1/query?query=up&time=2018-06-20T22:10:51.781Z -> vector
	output:

			http://localhost:9090/api/v1/query_range?query=up&start=2018-06-20T21:10:51.781Z&end=2018-06-20T22:10:51.781Z&step=15s -> matrix
	output:
	map[metric:map[__name__:up instance:localhost:9090 job:prometheus] values:[[1.529529051781e+09 1] [1.529529096781e+09 1] [1.529529141781e+09 1] [1.529529186781e+09 1] [1.529529231781e+09 1] [1.529529276781e+09 1] [1.529529321781e+09 1] [1.529529366781e+09 1] [1.529529411781e+09 1] [1.529529456781e+09 1] [1.529529501781e+09 1] [1.529529546781e+09 1] [1.529529591781e+09 1] [1.529529636781e+09 1]]]

	*/
	defer resp.Body.Close()
	if err == nil {
		body, err1 := ioutil.ReadAll(resp.Body)

		if err1 == nil {
			var prometheusResponse PrometheusResponse
			err2 := json.Unmarshal(body, &prometheusResponse)
			if err2 == nil {
				//fmt.Println(prometheusResponse)
				//data := &prometheusResponse.Data

				/*
								result := data.Result.([]interface{})[0]

								metric1 := result.(map[string]interface{})["metric"]
								//fmt.Println("metric1:", metric1)
								metric := Metric{__name__: metric1.(map[string]interface{})["__name__"].(string), Instance: metric1.(map[string]interface{})["instance"].(string), Job: metric1.(map[string]interface{})["job"].(string)}
				*/
				switch prometheusResponse.Data.ResultType {
				case "matrix":

					//for index, result := range data.Result.([]interface{}) {
					//
					//}

					//prometheusResponse.Data.Result = &MatrixResult{}
					//values1 := result.(map[string]interface{})["values"]
					//var values []Value
					//for _, currentValue := range values1.([]interface{}) {
					//	//fmt.Println("currentValue:", currentValue)
					//	sample, _ := strconv.Atoi(currentValue.([]interface{})[1].(string))
					//	value := Value{currentValue.([]interface{})[0].(float64), sample}
					//	//fmt.Println("value:", value)
					//	values = append(values, value)
					//}
					//
					////fmt.Println(reflect.TypeOf(values))
					//data.Result = MatrixResult{metric, values}
				case "vector":
					//fmt.Println(data.Result)
					//fmt.Println(data.Result.([]interface{}))
					//for _, result := range data.Result.([]interface{}) {
					//	//fmt.Println(result)
					//	metric := result.(map[string]interface{})["metric"]
					//	__name__ := metric.(map[string]interface{})["__name__"]
					//	value := result.(map[string]interface{})["value"]
					//	fmt.Println(__name__, value.([]interface{})[0], value.([]interface{})[0])
					//
					//	//value1 := result.(map[string]interface{})["value"]
					//	//sample, _ := strconv.Atoi(value1.([]interface{})[1].(string))
					//	//value := Value{value1.([]interface{})[0].(float64), sample}
					//	////fmt.Println(reflect.TypeOf(value))
					//	////fmt.Println(value)
					//	//data.Result = VectorResult{metric, value}
					//}

				case "scalar":

				case "string":
				}
				//fmt.Println(reflect.TypeOf(prometheusResponse.Data.Result))

				//json.Unmarshal(body, &prometheusResponse)
				//fmt.Println(reflect.TypeOf(prometheusResponse.Data.Result))

				//fmt.Println(prometheusResponse)
				return body
			} else {
				fmt.Print(err2, 57)
			}
			/*stringBody := string(body[:len(body)])
			fmt.Print(stringBody)*/
		} else {
			fmt.Print(err1, 62)
		}
	} else {
		fmt.Print(err, 65)
	}
	return []byte{}
}
