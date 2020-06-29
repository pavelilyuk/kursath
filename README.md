# kursath
import React from 'react';
import { Link } from 'react-router-dom';
import { getCartProducts } from '../repository';
import CartItem from './CartItem';

export default class Cart extends React.Component {
constructor(props) {
super(props);
this.state = {
products: [],
total: 0
}
}

componentWillMount() {
let cart = localStorage.getItem('cart');
if (!cart) return; 
getCartProducts(cart).then((products) => {
let total = 0;
for (var i = 0; i < products.length; i++) {
total += products[i].price * products[i].qty;
}
this.setState({ products, total });
});
}

removeFromCart = (product) => {
let products = this.state.products.filter((item) => item.id !== product.id);
let cart = JSON.parse(localStorage.getItem('cart'));
delete cart[product.id.toString()];
localStorage.setItem('cart', JSON.stringify(cart));
let total = this.state.total - (product.qty * product.price) 
this.setState({products, total});
}

clearCart = () => {
localStorage.removeItem('cart');
this.setState({products: []});
}

render() {
const { products, total } = this.state;
return (
<div className=" container">
<h3 className="card-title">Cart</h3>
<hr/>
{
products.map((product, index) => <CartItem product={product} remove={this.removeFromCart} key={index}/>)
}
<hr/>
{ products.length ? <div><h4><small>Total Amount:</small><span className="float-right text-primary">${total}</span></h4><hr/></div>: ''}

{ !products.length ? <h3 className="text-warning">No item on the cart</h3>: ''}
<Link to="/checkout"><button className="btn btn-success float-right">Checkout</button></Link>
<button className="btn btn-danger float-right" onClick={this.clearCart} style={{ marginRight: "10px" }}>Clear Cart</button>
<br/><br/><br/>
</div>
);
}
}
cart.js
import React from 'react';
import { login } from '../repository';

export default class Login extends React.Component{

constructor() {
super();
this.state = { name: '', password: '' };
this.handleInputChange =this.handleInputChange.bind(this);
this.submitLogin =this.submitLogin.bind(this);
}

handleInputChange(event) {
this.setState({[event.target.name]: event.target.value})
}

submitLogin(event){
event.preventDefault();
login(this.state)
.then(token => window.location = '/')
.catch(err => alert(err));
}

render() {
return (
<div className="container">
<hr/>
<div className="col-sm-8 col-sm-offset-2">
<div className="panel panel-primary">
<div className="panel-heading">
<h3>Log in </h3>
</div>
<div className="panel-body">
<form onSubmit={this.submitLogin}>
<div className="form-group">
<label>Name:</label>
<input type="text" className="form-control" name="name" onChange={this.handleInputChange}/>
</div>
<div className="form-group">
<label>Password:</label>
<input type="password" className="form-control" name="password" onChange={this.handleInputChange}/>
</div>
<button type="submit" className="btn btn-default">Submit</button>
</form>
</div>
</div>
</div>
</div>
);
}
}
login.js
import React, { Component } from 'react';
import Login from './components/Login';
import Products from './components/ProductList';
import Cart from './components/Cart';
import Checkout from './components/Checkout';
import { BrowserRouter as Router, Link, Route } from 'react-router-dom';
import { isAuthenticated } from './repository';


class App extends Component {

logOut(){
localStorage.removeItem('x-access-token');
}

render() {
const auth = isAuthenticated();
return (
<Router>
<div>
<nav className="navbar navbar-expand-lg navbar-dark bg-dark">
<div className="container">
<Link className="navbar-brand" to="/">Car Shop</Link>
<button className="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavAltMarkup" aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
<span className="navbar-toggler-icon"></span>
</button>
<div className="collapse navbar-collapse" id="navbarNavAltMarkup">
<div className="navbar-nav">
<Link className="nav-item nav-link" to="/">Products</Link>
<Link className="nav-item nav-link" to="/cart">Cart</Link>
{ (auth) ? <Link className="nav-item nav-link" to="/checkout">Checkout</Link>: ''}
{
( auth ) ? 
( <a className="nav-item nav-link" href="/" onClick={this.logOut}>Log out</a>) : 
( <Link className="nav-item nav-link float-right" to="/login">Log in</Link> )
}
</div>
</div>
</div>
</nav>
<div className="container">
<br/>
<Route exact path="/" component={Products} />
<Route exact path="/cart" component={Cart} />
<Route exact path="/checkout" component={Checkout} />
{ (!auth) ? <Route exact path="/login" component={Login} /> : '' }
</div>
</div>
</Router>
);
}
}

export default App;
app.js
import React from 'react';

export default class ProductItem extends React.Component {

constructor(props) {
super(props);
this.state = {
quantity: 1
}
}

handleInputChange = event => this.setState({ [event.target.name]: event.target.value })

addToCart = () => {
let cart = localStorage.getItem('cart') ? JSON.parse(localStorage.getItem('cart')) : {};
let id = this.props.product.id.toString();
cart[id] = (cart[id] ? cart[id] : 0);
let qty = cart[id] + parseInt(this.state.quantity);
if (this.props.product.available_quantity < qty) {
cart[id] = this.props.product.available_quantity;
} else {
cart[id] = qty
}
localStorage.setItem('cart', JSON.stringify(cart));
}

render() {
const { product } = this.props;
return (
<div className="card" style={{ marginBottom: "10px" }}>
<div className="card-body">
<h4 className="card-title">{product.name}</h4>
<p className="card-text">{product.description}</p>
<img src={this.props.imgSrc} alt="..." class="img-thumbnail img"></img>
<h5 className="card-text"><small>price: </small>${product.price}</h5>
<span className="card-text"><small>Available Quantity: </small>{product.available_quantity}</span>
{product.available_quantity > 0 ?
<div>
<button className="btn btn-sm btn-warning float-right" onClick={this.addToCart}>Add to cart</button>
<input type="number" value={this.state.quantity} name="quantity" onChange={this.handleInputChange} className="float-right" style={{ width: "60px", marginRight: "10px", borderRadius: "3px" }} />
</div> :
<p className="text-danger"> product is out of stock </p>
}
</div>
</div>
)
}
}
    product.js
import React from 'react';
import ProductItem from './ProductItem';
import { getProducts } from '../repository';
import { Link } from 'react-router-dom';
import item1 from '../img/item1.jpg';
import item2 from '../img/item2.jpg';
import item3 from '../img/item3.jpg';
import item4 from '../img/item4.jpg';

export default class ProductList extends React.Component {
constructor(props) {
super(props);
this.imgList = []
this.state = {
products: []
}
}

componentWillMount() {
getProducts().then((products) => {
this.setState({ products });
});
}
componentDidMount(){
this.imgList.push(item1);
this.imgList.push(item2);
this.imgList.push(item3);
this.imgList.push(item4);
}
render() { 
const { products } = this.state;
console.log(products)
return (
<div className=" container">
<h3 className="card-title">List of Available Products</h3>
<hr/>
{
products.map((product, index) => <ProductItem imgSrc={this.imgList[index]} product={product} key={index}/>)
}
<hr/>
<Link to="/checkout"><button className="btn btn-success float-right">Checkout</button></Link>
<Link to="/cart"><button className="btn btn-primary float-right" style={{ marginRight: "10px" }}>View Cart</button></Link>
<br/><br/><br/>
</div>
);
}
}
productList.js
import React from 'react';

export default class ProductItem extends React.Component {

constructor(props) {
super(props);
this.state = {
quantity: 1
}
}

handleInputChange = event => this.setState({ [event.target.name]: event.target.value })

addToCart = () => {
let cart = localStorage.getItem('cart') ? JSON.parse(localStorage.getItem('cart')) : {};
let id = this.props.product.id.toString();
cart[id] = (cart[id] ? cart[id] : 0);
let qty = cart[id] + parseInt(this.state.quantity);
if (this.props.product.available_quantity < qty) {
cart[id] = this.props.product.available_quantity;
} else {
cart[id] = qty
}
localStorage.setItem('cart', JSON.stringify(cart));
}

render() {
const { product } = this.props;
return (
<div className="card" style={{ marginBottom: "10px" }}>
<div className="card-body">
<h4 className="card-title">{product.name}</h4>
<p className="card-text">{product.description}</p>
<img src={this.props.imgSrc} alt="..." class="img-thumbnail img"></img>
<h5 className="card-text"><small>price: </small>${product.price}</h5>
<span className="card-text"><small>Available Quantity: </small>{product.available_quantity}</span>
{product.available_quantity > 0 ?
<div>
<button className="btn btn-sm btn-warning float-right" onClick={this.addToCart}>Add to cart</button>
<input type="number" value={this.state.quantity} name="quantity" onChange={this.handleInputChange} className="float-right" style={{ width: "60px", marginRight: "10px", borderRadius: "3px" }} />
</div> :
<p className="text-danger"> product is out of stock </p>
}
</div>
</div>
)
}
}
productItem.js
